---
title: "RemixをApp Runnerで動かす"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Remix", "AWS", "AppRunner", "Terraform"]
published: true
---

# 概要
Remixアプリケーションをコンテナで実行しTerraformを使ってApp Runnerで動かしてみたいと思います。

# Remixアプリケーション作成
npxコマンドでRemixアプリケーションを作成
```sh
npx create-remix@latest app
```

作成されたappディレクトリに移動し起動できることを確認
```sh
npm run dev
```

http://localhost:5173 にアクセスし以下の画面が表示されればOK
![remix dev](/images/remix.png)

# Dockerイメージの作成
Remixアプリケーションを実行するコンテナイメージを作成する
```Dockerfile: Dockerfile
# Build Container
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

RUN npm run build

# Run Container
FROM gcr.io/distroless/nodejs20-debian12 AS ruuner

WORKDIR /app

COPY --from=builder /app /app

USER nonroot

EXPOSE 3000

# distrolessイメージにはnpmが含まれていないため直接起動する
CMD ["node_modules/.bin/remix-serve", "./build/server/index.js"]
```

# AWSリソースの作成
Terraformで必要なAWSリソースを作成する
providerの指定はこんな感じ
```tf: provider.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "ap-northeast-1"
}
```

Terraformを初期化する
```sh
terraform init
```

まずはコンテナをプッシュするECRを作成する
```tf: ecr.tf
resource "aws_ecr_repository" "remix-app" {
  name                 = "remix-app"
  image_tag_mutability = "MUTABLE"
  force_delete         = true # 最後にリソースをすべて削除するため

  image_scanning_configuration {
    scan_on_push = true
  }
}
```

コンテナリポジトリの作成
```sh
terraform apply -target=aws_ecr_repository.remix-app
```

作成したリポジトリにDockerイメージをpush
コマンドはAWSのリポジトリを参照

App Runnerを作成する
同時にIAMロールも作成し作成したリポジトリにアクセスできるようにする
```tf: iam.tf
resource "aws_iam_role" "app-runner-role" {
  name = "app-runner-role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": [
          "build.apprunner.amazonaws.com",
          "tasks.apprunner.amazonaws.com"
        ]
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "app-runner-role-attachment" {
  role       = aws_iam_role.app-runner-role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSAppRunnerServicePolicyForECRAccess"
}
```
```tf: apprunner.tf
resource "aws_apprunner_service" "remix-app" {
  service_name = "remix-app"

  source_configuration {
    authentication_configuration {
      access_role_arn = aws_iam_role.app-runner-role.arn
    }
    image_repository {
      image_identifier      = "${aws_ecr_repository.remix-app.repository_url}:latest"
      image_repository_type = "ECR"
      image_configuration {
        port = 3000
      }
    }
    auto_deployments_enabled = true
  }

  tags = {
    Name = "remix-app"
  }
}
```

また、アクセスするURLを知りたいので取得できるようにする
```tf: output.tf
output "app_runner_url" {
  value = aws_apprunner_service.remix-app.service_url
}
```

AppRunnerを構築する
```sh
terraform apply
```

表示されたURLにアクセスしてページが表示されれば成功
![remix app](/images/remix.png)

最後に作成したリソースを削除しておく
```sh
terraform destroy
```
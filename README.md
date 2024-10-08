# Golang GitHub Action Template

このリポジトリは、Golangプロジェクトをビルドし、AWS ECRにDockerイメージをプッシュし、Lambda関数としてデプロイするためのテンプレートです。GitHub Actionsを使用して、CI/CDパイプラインを実行します。

## 目次

1. [必要要件](#必要要件)
2. [セットアップ](#セットアップ)
3. [GitHub Actions ワークフローの説明](#github-actions-ワークフローの説明)
4. [CloudFormation テンプレートの概要](#cloudformation-テンプレートの概要)
5. [ローカル開発とテスト](#ローカル開発とテスト)

## 必要要件

このプロジェクトを実行するには、以下のツールが必要です。

- [AWS CLI](https://aws.amazon.com/cli/)
- [Docker](https://www.docker.com/)
- [Golang](https://golang.org/)
- [GitHub CLI](https://cli.github.com/)

## セットアップ

1. リポジトリをクローンします。

    ```bash
    git clone https://github.com/your-repo/golang-github-action-template.git
    cd golang-github-action-template
    ```

2. 必要なAWSリソースを作成します（ECR、IAMロールなど）。

    - Cognitoを使う場合、Cognitoユーザープールやアプリクライアントを設定してください。

3. 環境変数をGitHub Secretsに設定します。

   GitHubのリポジトリ設定から以下のシークレットを追加します。

    - `AWS_REGION`: AWSリージョン
    - `AWS_ROLE_TO_ASSUME`: AWS IAMロール
    - `AWS_ACCOUNT_ID`: AWSアカウントID
    - `AWS_ACCESS_KEY_ID`: AWSアクセスキー
    - `AWS_SECRET_ACCESS_KEY`: AWSシークレットキー

## GitHub Actions ワークフローの説明

このリポジトリには、以下のGitHub Actionsワークフローが含まれています。

### `.github/workflows/main.yml`

- **目的**: プッシュイベントがトリガーされた際に、リポジトリの変更に基づいてDockerイメージをビルドし、AWS ECRにプッシュし、CloudFormationを使用してLambda関数をデプロイします。

- **主なステップ**:

    1. リポジトリ名の抽出
    2. AWS ECRリポジトリの存在確認と作成
    3. ECRリポジトリへのDockerイメージのプッシュ
    4. CloudFormationを使用してLambda関数をデプロイ

### 実行例

```yaml
name: ECR Push Image with Commit Message

on:
  push:
    branches:
      - 'main'

jobs:
  extract-commit-message:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: |
          echo "リポジトリ名を抽出"
  push:
    runs-on: ubuntu-latest
    needs: extract-commit-message
    steps:
      - uses: actions/checkout@v3
      - run: |
          echo "ECRリポジトリにイメージをプッシュ"
  deploy:
    runs-on: ubuntu-latest
    needs: push
    steps:
      - uses: actions/checkout@v3
      - run: |
          echo "CloudFormationを使用してLambdaをデプロイ"
  ```

## CloudFormation テンプレートの概要

`cloudformation/cloudformation-template.yaml` では、ECRから取得したDockerイメージを使用してLambda関数をデプロイします。

- **MyLambdaFunction**: ECRにプッシュされたDockerイメージを元にLambda関数を作成します。
- **LambdaExecutionRole**: Lambda関数がECRからイメージを取得するために必要なIAMロールを作成します。

## ローカル開発とテスト

### Dockerイメージのビルド

ローカルでDockerイメージをビルドするには、以下のコマンドを実行します。

```bash
docker-compose build
```

### Lambda関数のローカルテスト

AWS SAM CLIを使用して、ローカルでLambda関数をテストできます。

1. SAMプロジェクトをビルドします。

```bash
sam build --template-file cloudformation/local-template.yaml
```

2. ローカルでLambda関数を実行します。

```bash
sam local invoke MyLambdaFunction --template-file cloudformation/local-template.yaml
```

### イベントペイロードを使用したテスト

特定のイベントペイロードでLambda関数をテストするには、`event.json` ファイルを作成し、以下のコマンドを実行します。

```bash
sam local invoke MyLambdaFunction --template-file cloudformation/local-template.yaml --event event.json
```

## デプロイ

GitHub Actionsによって自動的にデプロイが行われますが、手動でデプロイする場合は以下のコマンドを使用できます。

```bash
sam deploy --guided
```

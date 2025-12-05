<!--
  <<< Author notes: Step 5 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

## Step 5: label に基づいて production 環境にデプロイする

_デプロイされました! :ship:_

### よくできました

前に行ったように、`main` から `production-deployment-workflow` という名前の新しい branch を作成します。`.github/workflows` ディレクトリに、`deploy-prod.yml` という名前の新しいファイルを追加します。この新しいワークフローは、`main` への commit を特に処理し、`prod` へのデプロイを処理します。

**継続的デリバリー** (CD) は、多くの振る舞いやより具体的な概念を含む概念です。そのような概念の1つに **production 環境でのテスト**があります。これはプロジェクトや企業によって異なる意味を持ち、CD を「行っている」かどうかを定める厳密なルールではありません。

私たちの場合、production 環境を staging 環境と完全に同じように合わせることができます。これにより、production にデプロイした後の予期せぬ事態の機会が最小限になります。

### :keyboard: Activity 1: production デプロイメントワークフローにトリガーを追加する

以下をファイルにコピー＆ペーストし、`<username>` プレースホルダーを自分の GitHub ユーザー名に置き換えてください。staging ワークフローからあまり変更されていないことに注意してください。トリガーと label でフィルタリングしない点が異なります。

```yaml
name: Deploy to production

on:
  push:
    branches:
      - main

env:
  IMAGE_REGISTRY_URL: ghcr.io
  ###############################################
  ### <username> を GitHub ユーザー名に置き換える ###
  ###############################################
  DOCKER_IMAGE_NAME: <username>-azure-ttt
  AZURE_WEBAPP_NAME: <username>-ttt-app
  ###############################################

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 16
      - name: npm install and build webpack
        run: |
          npm install
          npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: webpack artifacts
          path: public/

  Build-Docker-Image:
    runs-on: ubuntu-latest
    needs: build
    name: Build image and store in GitHub Container Registry
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download built artifact
        uses: actions/download-artifact@v4
        with:
          name: webpack artifacts
          path: public

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.IMAGE_REGISTRY_URL }}
          username: ${{ github.actor }}
          password: ${{ secrets.CR_PAT }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}
          tags: |
            type=sha,format=long,prefix=

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  Deploy-to-Azure:
    runs-on: ubuntu-latest
    needs: Build-Docker-Image
    name: Deploy app container to Azure
    steps:
      - name: "Login via Azure CLI"
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - uses: azure/docker-login@v1
        with:
          login-server: ${{env.IMAGE_REGISTRY_URL}}
          username: ${{ github.actor }}
          password: ${{ secrets.CR_PAT }}

      - name: Deploy web app container
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{env.AZURE_WEBAPP_NAME}}
          images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}:${{github.sha}}

      - name: Azure logout via Azure CLI
        uses: azure/CLI@v2
        with:
          inlineScript: |
            az logout
            az cache purge
            az account clear
```

1. すべての `<username>` を自分の GitHub ユーザー名に更新します。
1. 変更を `production-deployment-workflow` branch に commit します。
1. Pull requests タブに移動し、`production-deployment-workflow` branch の **Compare & pull request** をクリックして Pull request を作成します。

素晴らしい!使用した構文は、GitHub Actions に対して main branch に commit が行われたときにのみワークフローを実行するように指示します。これで、このワークフローを実際に使用して production にデプロイできます!

### :keyboard: Activity 2: pull request を merge する

1. これで pull request を [merge](https://docs.github.com/get-started/quickstart/github-glossary#merge) できます!
1. **Merge pull request** をクリックし、次のステップで閉じた pull request に label を適用するため、このタブを開いたままにしておきます。
1. あとは、パッケージが GitHub Container Registry に公開され、デプロイが実行されるのを待つだけです。
1. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

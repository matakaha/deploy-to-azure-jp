<!--
  <<< Author notes: Step 2 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

## Step 2: Azure 環境をセットアップする

_始められましたね、お疲れさまです :gear:_

### 特定の label で job をトリガーすることができるようになりました

このワークフローのステップについて詳しく説明はしませんが、使用しているアクションについて理解しておくことをお勧めします。使用しているアクションは次のとおりです:

- [`actions/checkout`](https://github.com/actions/checkout)
- [`actions/upload-artifact`](https://github.com/actions/upload-artifact)
- [`actions/download-artifact`](https://github.com/actions/download-artifact)
- [`docker/login-action`](https://github.com/docker/login-action)
- [`docker/build-push-action`](https://github.com/docker/build-push-action)
- [`azure/login`](https://github.com/Azure/login)
- [`azure/webapps-deploy`](https://github.com/Azure/webapps-deploy)

### :keyboard: Activity 1: 認証情報を GitHub secrets に保存し、ワークフローのセットアップを完了する

1.  新しいタブで、まだアカウントをお持ちでない場合は [Azure アカウントを作成](https://azure.microsoft.com/ja-jp/free/)してください。Azure アカウントが職場で作成されている場合、必要なリソースへのアクセスに問題が発生する可能性があります。個人用に新しいアカウントを作成し、このコースで使用することをお勧めします。
    > **注**: Azure アカウントを作成するにはクレジットカードが必要な場合があります。学生の方は、Azure へのアクセスのために [Student Developer Pack](https://education.github.com/pack) を利用できる場合があります。Azure アカウントなしでコースを続けたい場合、Skills は引き続き応答しますが、デプロイは機能しません。
1.  Azure Portal で[新しい subscription を作成](https://docs.microsoft.com/ja-jp/azure/cost-management-billing/manage/create-subscription)します。
    > **注**: subscription は「従量課金制」に設定する必要があり、請求情報を入力する必要があります。このコースでは無料プランから数分しか使用しませんが、Azure では請求情報が必要です。
1.  マシンに [Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。
1.  ターミナルで次のコマンドを実行します:
    ```shell
    az login
    ```
1.  対話型認証プロンプトから先ほど選択した subscription を選択します。subscription ID の値を安全な場所にコピーします。これを `AZURE_SUBSCRIPTION_ID` と呼びます。例は次のようになります:
    ```shell
    No     Subscription name    Subscription ID                       Tenant
    -----  -------------------  ------------------------------------  -----------------
    [1] *  some-subscription    f****a09-****-4d1c-98**-f**********c  Default Directory
    ```
1.  ターミナルで以下のコマンドを実行します。

    ```shell
    az ad sp create-for-rbac --name "GitHub-Actions" --role contributor \
     --scopes /subscriptions/{subscription-id} \
     --sdk-auth

        # {subscription-id} を AZURE_SUBSCRIPTION_ID に保存した同じ ID に置き換えてください。
    ```

    > **注**: `\` 文字は Unix ベースのシステムで改行として機能します。Windows ベースのシステムを使用している場合、`\` 文字はこのコマンドを失敗させます。Windows を使用している場合は、このコマンドを1行に配置してください。

1.  コマンドの応答の内容全体をコピーします。これを `AZURE_CREDENTIALS` と呼びます。例は次のようになります:
    ```shell
    {
      "clientId": "<GUID>",
      "clientSecret": "<GUID>",
      "subscriptionId": "<GUID>",
      "tenantId": "<GUID>",
      (...)
    }
    ```
1.  GitHub に戻り、このリポジトリの Settings タブの **Secrets and variables > Actions** をクリックします。
1.  **New repository secret** をクリックします。
1.  新しい secret に **AZURE_SUBSCRIPTION_ID** という名前を付け、最初のコマンドの `id:` フィールドの値を貼り付けます。
1.  **Add secret** をクリックします。
1.  再度 **New repository secret** をクリックします。
1.  2つ目の secret に **AZURE_CREDENTIALS** という名前を付け、入力した2つ目のターミナルコマンドの内容全体を貼り付けます。
1.  **Add secret** をクリックします。
1.  Pull requests タブに戻り、pull request の **Files Changed** タブに移動します。`.github/workflows/deploy-staging.yml` ファイルを見つけて編集し、新しいアクションを使用します。完全なワークフローファイルは次のようになります:
    ```yaml
    name: Deploy to staging

    on:
      pull_request:
        types: [labeled]

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
        if: contains(github.event.pull_request.labels.*.name, 'stage')

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
              images: ${{env.IMAGE_REGISTRY_URL}}/${{ github.repository }}/${{env.DOCKER_IMAGE_NAME}}:${{ github.sha }}

          - name: Azure logout via Azure CLI
            uses: azure/CLI@v2
            with:
              inlineScript: |
                az logout
                az cache purge
                az account clear
    ```
1. ファイルを編集したら、**Commit changes...** をクリックし、`staging-workflow` branch に commit します。
1. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

<!--
  <<< Author notes: Step 3 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

## Step 3: label に基づいて環境を起動する

_よくできました! :heart:_

GitHub Actions はクラウドに依存しないため、どのクラウドでも機能します。このコースでは Azure へのデプロイ方法を紹介します。

**_Azure リソース_とは何ですか?** Azure では、リソースは Azure によって管理されるエンティティです。このコースでは、以下の Azure リソースを使用します:

- [web app](https://docs.microsoft.com/ja-jp/azure/app-service/overview) は、アプリケーションを Azure にデプロイする方法です。
- [resource group](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/management/overview#resource-groups) は、web app や仮想マシン (VM) などのリソースのコレクションです。
- [App Service plan](https://docs.microsoft.com/ja-jp/azure/app-service/overview-hosting-plans) は、web app を実行し、課金を管理するものです(このアプリは無料で実行されるはずです)。

GitHub Actions の力により、ワークフローファイルを通じてこれらのリソースを作成、設定、削除することができます。

### :keyboard: Activity 1: personal access token (PAT) をセットアップする

Personal access token (PAT) は、GitHub への認証にパスワードを使用する代わりの方法です。ワークフローが新しくビルドされたイメージを registry にプッシュした後、web app が container イメージをプルできるようにするために PAT を使用します。

1. 新しいブラウザタブを開き、このタブで指示を読みながら、2つ目のタブで手順を進めてください。
2. `repo` と `write:packages` スコープを持つ personal access token を作成します。詳細については、[「personal access token の作成」](https://docs.github.com/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)を参照してください。
3. トークンを生成したら、ワークフロー内で使用できるように secret に保存する必要があります。`CR_PAT` という名前の新しい repository secret を作成し、PAT トークンを値として貼り付けます。
4. これで、ワークフローのセットアップに進むことができます。

**Azure 環境の設定**

Azure 環境に正常にデプロイするには:

1. リポジトリページの `Code` タブの左上にある branch ドロップダウンをクリックして、`azure-configuration` という名前の新しい branch を作成します。
2. 新しい `azure-configuration` branch に入ったら、`.github/workflows` ディレクトリに移動し、**Add file** をクリックして `spinup-destroy.yml` という名前の新しいファイルを作成します。以下を新しいファイルにコピー＆ペーストします:
    ```yaml
    name: Configure Azure environment

    on:
      pull_request:
        types: [labeled]

    env:
      IMAGE_REGISTRY_URL: ghcr.io
      AZURE_RESOURCE_GROUP: cd-with-actions
      AZURE_APP_PLAN: actions-ttt-deployment
      AZURE_LOCATION: '"East US"'
      ###############################################
      ### <username> を GitHub ユーザー名に置き換える ###
      ###############################################
      AZURE_WEBAPP_NAME: <username>-ttt-app

    jobs:
      setup-up-azure-resources:
        runs-on: ubuntu-latest
        if: contains(github.event.pull_request.labels.*.name, 'spin up environment')
        steps:
          - name: Checkout repository
            uses: actions/checkout@v4

          - name: Azure login
            uses: azure/login@v2
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}

          - name: Create Azure resource group
            if: success()
            run: |
              az group create --location ${{env.AZURE_LOCATION}} --name ${{env.AZURE_RESOURCE_GROUP}} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

          - name: Create Azure app service plan
            if: success()
            run: |
              az appservice plan create --resource-group ${{env.AZURE_RESOURCE_GROUP}} --name ${{env.AZURE_APP_PLAN}} --is-linux --sku F1 --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

          - name: Create webapp resource
            if: success()
            run: |
              az webapp create --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --plan ${{ env.AZURE_APP_PLAN }} --name ${{ env.AZURE_WEBAPP_NAME }}  --deployment-container-image-name nginx --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

          - name: Configure webapp to use GHCR
            if: success()
            run: |
              az webapp config container set --docker-custom-image-name nginx --docker-registry-server-password ${{secrets.CR_PAT}} --docker-registry-server-url https://${{env.IMAGE_REGISTRY_URL}} --docker-registry-server-user ${{github.actor}} --name ${{ env.AZURE_WEBAPP_NAME }} --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

      destroy-azure-resources:
        runs-on: ubuntu-latest

        if: contains(github.event.pull_request.labels.*.name, 'destroy environment')

        steps:
          - name: Checkout repository
            uses: actions/checkout@v4

          - name: Azure login
            uses: azure/login@v2
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}

          - name: Destroy Azure environment
            if: success()
            run: |
              az group delete --name ${{env.AZURE_RESOURCE_GROUP}} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}} --yes
    ```
1. **Commit changes...** をクリックし、**Commit changes** をクリックする前に `Commit directly to the azure-configuration branch.` を選択します。
1. リポジトリの Pull requests タブに移動します。
1. `azure-configuration` branch とともに黄色いバナーが表示されるので、**Compare & pull request** をクリックします。
1. Pull request のタイトルを: `Added spinup-destroy.yml workflow` に設定し、`Create pull request` をクリックします。

主要な機能を以下で説明し、その後 pull request に label を適用してワークフローを実際に使用します。

この新しいワークフローには2つの job があります:

1. **Set up Azure resources** は、pull request に "spin up environment" という名前の label が含まれている場合に実行されます。
2. **Destroy Azure resources** は、pull request に "destroy environment" という名前の label が含まれている場合に実行されます。

各 job に加えて、いくつかのグローバル環境変数があります:

- `AZURE_RESOURCE_GROUP`、`AZURE_APP_PLAN`、`AZURE_WEBAPP_NAME` は、複数のステップとワークフローで参照する resource group、App Service plan、web app の名前です。
- `AZURE_LOCATION` は、アプリが最終的にデプロイされるデータセンターの[リージョン](https://azure.microsoft.com/ja-jp/explore/global-infrastructure/geographies/#geographies)を指定します。

**Azure リソースのセットアップ**

最初の job は次のように Azure リソースをセットアップします:

1. [`azure/login`](https://github.com/Azure/login) アクションを使用して Azure アカウントにログインします。以前に作成した `AZURE_CREDENTIALS` secret が認証に使用されます。
1. Azure CLI で [`az group create`](https://docs.microsoft.com/ja-jp/cli/azure/group?view=azure-cli-latest#az-group-create) を実行して [Azure resource group](https://docs.microsoft.com/ja-jp/azure/azure-resource-manager/management/overview#resource-groups) を作成します。Azure CLI は [GitHub ホスト型 runner にプリインストール](https://docs.github.com/actions/concepts/runners/github-hosted-runners)されています。
1. Azure CLI で [`az appservice plan create`](https://docs.microsoft.com/ja-jp/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) を実行して [App Service plan](https://docs.microsoft.com/ja-jp/azure/app-service/overview-hosting-plans) を作成します。
1. Azure CLI で [`az webapp create`](https://docs.microsoft.com/ja-jp/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) を実行して [web app](https://docs.microsoft.com/ja-jp/azure/app-service/overview) を作成します。
1. Azure CLI で [`az webapp config`](https://docs.microsoft.com/ja-jp/cli/azure/webapp/config?view=azure-cli-latest) を使用して、新しく作成した web app が [GitHub Packages](https://docs.github.com/packages/learn-github-packages/introduction-to-github-packages) を使用するように設定します。Azure は、独自の [Azure Container Registry](https://docs.microsoft.com/ja-jp/azure/container-registry/)、[DockerHub](https://docs.docker.com/docker-hub/)、またはカスタム(プライベート) registry を使用するように設定できます。今回は、GitHub Packages をカスタム registry として設定します。

**Azure リソースの削除**

2番目の job は、無料分を消費したり課金が発生したりしないように Azure リソースを削除します。job は次のように動作します:

1. [`azure/login`](https://github.com/Azure/login) アクションを使用して Azure アカウントにログインします。以前に作成した `AZURE_CREDENTIALS` secret が認証に使用されます。
1. Azure CLI で [`az group delete`](https://docs.microsoft.com/ja-jp/cli/azure/group?view=azure-cli-latest#az-group-delete) を使用して、以前に作成した resource group を削除します。

### :keyboard: Activity 2: リソースを作成するために label を適用する

1. 開いている pull request の `spinup-destroy.yml` ファイルを編集し、`<username>` プレースホルダーを自分の GitHub ユーザー名に置き換えます。この変更を `azure-configuration` branch に直接 commit します。
1. Pull request に戻り、開いている pull request に `spin up environment` label を作成して適用します。
1. GitHub Actions ワークフローが実行され、Azure 環境が起動されるのを待ちます。Actions タブまたは pull request の merge ボックスで進行状況を確認できます。
1. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

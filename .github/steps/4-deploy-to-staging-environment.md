<!--
  <<< Author notes: Step 4 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

## Step 4: label に基づいて staging 環境にデプロイする

_よくできました。ワークフローを使用して Azure 環境を起動しましたね :dancer:_

適切な設定とワークフローファイルが揃ったので、アクションをテストしましょう!このステップでは、ゲームに少し変更があります。適切な label を pull request に追加すると、デプロイを確認できるはずです!

1. 前の `azure-configuration` branch と同じ手順で、`main` から `staging-test` という名前の新しい branch を作成します。
1. `.github/workflows/deploy-staging.yml` ファイルを編集し、すべての `<username>` を自分の GitHub ユーザー名に置き換えます。
1. その変更を新しい `staging-test` branch に commit します。
1. Pull requests タブに移動すると、`staging-test` branch の黄色いバナーが表示されるので、`Compare & pull request` をクリックします。pull request が開いたら、`Create pull request` をクリックします。

### :keyboard: Activity 1: pull request に適切な label を追加する

1. このリポジトリの `GITHUB_TOKEN` が **Workflow permissions** の下で読み取りと書き込みの権限を持っていることを確認します。[詳細はこちら](https://docs.github.com/actions/security-guides/automatic-token-authentication#modifying-the-permissions-for-the-github_token)。これは、ワークフローが container registry にイメージをアップロードできるようにするために必要です。
1. 開いている pull request に `stage` label を作成して適用します。
1. GitHub Actions ワークフローが実行され、アプリケーションが Azure 環境にデプロイされるのを待ちます。Actions タブまたは pull request の merge ボックスで進行状況を確認できます。デプロイには数分かかる場合がありますが、正しく行われています。デプロイが成功すると、各実行に緑色のチェックマークが表示され、デプロイの URL が表示されます。ゲームをプレイしてみましょう!
1. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

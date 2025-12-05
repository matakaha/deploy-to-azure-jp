<!--
  <<< Author notes: Step 6 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
-->

## Step 6: Production デプロイメント

_よくできました! :sparkle:_

素晴らしい作業です、やり遂げました!アカウントの **Packages** セクション(メインのリポジトリページ)で container イメージを確認できるはずです。デプロイメント URL は、staging URL と同様に Actions ログで取得できます。

### クラウド環境

コース全体を通じて、放置すると課金が発生したり、クラウドプロバイダーからの無料分を消費したりする可能性のあるリソースを起動してきました。アプリケーションが production で確認できたら、これらの環境を削除して、学習のための時間を確保できるようにしましょう!

### :keyboard: Activity 1: 実行中のリソースを削除して課金が発生しないようにする

1. merge した `production-deployment-workflow` pull request に `destroy environment` label を作成して適用します。pull request を閉じたタブを既に閉じている場合は、**Pull requests** をクリックし、**Closed** フィルターをクリックして merge された pull request を表示することで、再度開くことができます。

適切な label を適用したので、GitHub Actions ワークフローが完了するのを待ちましょう。完了したら、アプリの URL にアクセスするか、Azure portal にログインして実行されていないことを確認することで、環境が削除されたことを確認できます。

2. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

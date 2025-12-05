<!--
  <<< Author notes: Step 1 >>>
  Choose 3-5 steps for your course.
  The first step is always the hardest, so pick something easy!
  Link to docs.github.com for further explanations.
  Encourage users to open new tabs for steps!
-->

## Step 1: label に基づいて job をトリガーする

_コースへようこそ :tada:_

![Screen Shot 2022-06-07 at 4 01 43 PM](https://user-images.githubusercontent.com/6351798/172490466-00f27580-8906-471f-ae83-ef3b6370df7d.png)

「継続的に」デリバリーするには、多くのことが必要です。これらは文化や行動から特定の自動化まで多岐にわたります。この演習では、自動化のデプロイ部分に焦点を当てます。

GitHub Actions ワークフローでは、`on` ステップでワークフローを実行するトリガーを定義します。今回は、pull request に特定の label が適用されたときに、ワークフローが異なるタスクを実行するようにします。

label をトリガーとして複数のタスクに使用します:

- 誰かが pull request に "spin up environment" という label を適用すると、GitHub Actions に対して Azure 環境でリソースをセットアップしたいことを伝えます。
- 誰かが pull request に "stage" という label を適用すると、アプリケーションを staging 環境にデプロイしたいことを示します。
- 誰かが pull request に "destroy environment" という label を適用すると、Azure アカウントで実行されているすべてのリソースを削除します。

### :keyboard: Activity 1: `GITHUB_TOKEN` の権限を設定する

各ワークフロー実行の開始時に、GitHub は自動的に一意の `GITHUB_TOKEN` secret を作成し、ワークフロー内で使用できるようにします。このトークンがこのコースに必要な権限を持っていることを確認する必要があります。

1. 新しいブラウザタブを開き、2つ目のタブで手順を進めながら、このタブで指示を読んでください。
1. Settings > Actions > General に移動します。**Workflow permissions** の下で、`GITHUB_TOKEN` が **Read and write permissions** を有効にしていることを確認してください。これは、ワークフローが container registry にイメージをアップロードするために必要です。

### :keyboard: Activity 2: label に基づくトリガーを設定する

今のところ、staging に焦点を当てます。環境の起動と削除は後のステップで行います。

1. **Actions** タブに移動します。
1. **New workflow** をクリックします。
1. "simple workflow" を検索し、**Configure** をクリックします。
1. ワークフローに `deploy-staging.yml` という名前を付けます。
1. このファイルの内容を編集し、すべてのトリガーと job を削除します。
1. ファイルの内容を編集して、**stage** という label が存在する場合に `build` job をフィルタリングする条件を追加します。最終的なファイルは次のようになります:
   ```yaml
   name: Stage the app

   on:
     pull_request:
       types: [labeled]

   jobs:
     build:
       runs-on: ubuntu-latest

       if: contains(github.event.pull_request.labels.*.name, 'stage')
   ```
1. **Commit Changes...** をクリックし、`staging-workflow` という名前の新しい branch を作成すること(Create new branch for this commit and start a pull request)を選択します。
1. **Propose changes** をクリックします。
1. **Create pull request** をクリックします。
1. 約20秒待ってから、このページ(指示を読んでいるページ)を更新してください。[GitHub Actions](https://docs.github.com/actions) が自動的に次のステップに更新されます。

<header>

<!--
  <<< Author notes: Course header >>>
  Include a 1280x640 image, course title in sentence case, and a concise description in emphasis.
  In your repository settings: enable template repository, add your 1280x640 social image, auto delete head branches.
  Add your open source license, GitHub uses MIT license.
-->

# Azure へのデプロイ

_GitHub Actions と Microsoft Azure を使用して2つのデプロイワークフローを作成します。_

</header>

<!--
  <<< Author notes: Course start >>>
  Include start button, a note about Actions minutes,
  and tell the learner why they should take the course.
-->

## ようこそ

GitHub Actions と Microsoft Azure を使用して2つのデプロイワークフローを作成します。

- **対象者**: 開発者、DevOps エンジニア、GitHub 初心者、学生、チーム
- **学習内容**: GitHub Actions と Microsoft Azure を使用して継続的デリバリー (Continuous Delivery) を実現するワークフローの作成方法を学びます。
- **作成するもの**: 2つのデプロイワークフローを作成します。1つ目は label に基づいて staging 環境にデプロイするワークフロー、2つ目は main への merge に基づいて production 環境にデプロイするワークフローです。
- **前提条件**: GitHub、GitHub Actions、GitHub Actions を使用した継続的インテグレーション (Continuous Integration) について理解していることが望ましいです。
- **所要時間**: このコースは2時間未満で完了できます。

このコースでは、以下のことを学びます:

1. job の設定
2. Azure 環境のセットアップ
3. 環境の起動
4. staging へのデプロイ
5. production へのデプロイ
6. 環境の削除

### このコースの始め方

<!-- For start course, run in JavaScript:
'https://github.com/new?' + new URLSearchParams({
  template_owner: 'skills',
  template_name: 'deploy-to-azure',
  owner: '@me',
  name: 'skills-deploy-to-azure',
  description: 'My clone repository',
  visibility: 'public',
}).toString()
-->

[![start-course](https://user-images.githubusercontent.com/1221423/235727646-4a590299-ffe5-480d-8cd5-8194ea184546.svg)](https://github.com/new?template_owner=matakaha&template_name=deploy-to-azure-jp&owner=%40me&name=skills-deploy-to-azure-jp&description=My+clone+repository&visibility=public)

1. **Start course** を右クリックして、新しいタブでリンクを開きます。
2. 新しいタブでは、ほとんどの項目が自動的に入力されます。
   - owner には、個人アカウントまたは organization を選択してリポジトリをホストします。
   - private リポジトリは [Actions の実行時間を消費する](https://docs.github.com/billing/managing-billing-for-github-actions/about-billing-for-github-actions)ため、public リポジトリを作成することを推奨します。
   - 下にスクロールして、フォーム下部の **Create repository** ボタンをクリックします。
3. 新しいリポジトリが作成されたら、約20秒待ってからページを更新してください。新しいリポジトリの README にあるステップバイステップの手順に従ってください。

<footer>

<!--
  <<< Author notes: Footer >>>
  Add a link to get support, GitHub status page, code of conduct, license link.
-->

---

ヘルプ: [ディスカッションボードに投稿](https://github.com/orgs/skills/discussions/categories/deploy-to-azure) &bull; [GitHub status ページを確認](https://www.githubstatus.com/)

&copy; 2023 GitHub &bull; [行動規範](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

</footer>

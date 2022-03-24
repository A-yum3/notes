## GitHub PR用templateを追加する

https://docs.github.com/ja/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository

### ドキュメント要約フロー

1. テンプレートを追加したいリポジトリで適当にブランチを切る
2. templateになるmarkdown形式のファイルを作成する（場所はいずれか）
	1. `/pull_request_template.md`という形式
	2. `docs/pull_request_template.md`という形式
	3. `.github/upp_request_template.md`という形式
3. 複数形式を使いたい場合は`./github/PULL_REQUEST_TEMPLATE/pull_request_template.md`のように`PULL_REQUEST_TEMPLATE`サブディレクトリを作ることで可能
4. コミット&マージする

### 備考

- Issueも同様にテンプレートを作成できる

### テンプレートを作る際の参考

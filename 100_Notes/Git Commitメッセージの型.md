#git 

ref: https://times.hrbrain.co.jp/entry/how-to-write-commit-message-type

- fix
	- コードベースのバグにパッチを当てる
- feat
	-  新しい機能を追加する
- BREAKING CHANGE型
	- フッターにBREAKING CHANGE:が書かれている
	- 型/スコープの直後に「！」が追加されているコミットは破壊的変更を導入する
- docs
	- ドキュメントのみの変更
- style
	- フォーマットの変更(コードの動作に影響しない範囲)
- refactor
	- リファクタリングのための変更
- perf
	- パフォーマンス改善のための変更
- test
	- 不足テストの追加や既存テストの修正
- build
	- ビルドシステムや外部依存に関する変更
- ci
	- CI用の設定やスクリプトに関する変更
- chore
	- その他の変更（ソースやテストの変更を含まない）
- revert
	- 以前のコミットに復帰

コミットの型には追加の文脈情報としてスコープを追加することができる。スコープは（）で囲む。`feat(parser): add ability to parse arrays`
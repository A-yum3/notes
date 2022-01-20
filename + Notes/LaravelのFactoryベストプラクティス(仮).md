#php/laravel/test 

[[Factoryの運用方法]]から

- Factoryの定義をする
	- Factoryはfakerなどを使って、factoryメソッドだけで問題なくfactoryされるようにする
	- 外部キーは `SampleModel::facotry()`でidを決め打ちしない
- stateメソッドを基本的に肥やして使う時はメソッドチェーン可能にする
	- stateで行うのは単一属性を変更する程度に留める
	- return値はオブジェクトにする
- state以外のデータセットを作成するメソッドはオリジナルFactoryを作成してもいいかも(案)
	- DataSetsクラスを作る？

メリット
- 変更が出た時に1箇所で済む
- 可読性があがる（ちゃんとコメントを書いて、メソッド名を書けば）
- 将来的なコストは低くなる（ヘルパーメソッドを選択するだけ）

デメリット
- 手間が増える
- Factoryはコード補完が効かない

対策
- Factoryをラップして自作Factoryを作る
	- https://github.com/spatie/freek.dev/blob/main/tests/Factories/PostFactory.php

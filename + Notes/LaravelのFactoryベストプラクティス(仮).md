#php/laravel/test 

- Factoryの定義は必須のものだけ
- stateメソッドを基本的に肥やして使う時はメソッドチェーン
	- return値はオブジェクトにする
- 外部キーの状態を変えたかったら外部キー側のstate

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

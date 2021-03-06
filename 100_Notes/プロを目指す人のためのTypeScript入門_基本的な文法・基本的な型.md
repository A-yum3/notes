## 文と式

- 文と式の決定的な違いは、**結果があるかどうか**
- 文は宣言とその他に分かれる
- 式文
	- 式のあとにセミコロンを書く文
		- `console.log(greeting + target);`

## 変数の宣言と使用

- `const 変数名 = 式;`
- カンマで宣言はつなげられる
	- `const greeting = "Hello, ", ...`
		- 可読性が劣るので人気がなく主流でない
- 型注釈
	- `const 変数:型 = 式;`
- letは再代入可能、宣言時に値を代入しなくてもよい
	- letは避ける
	- 極力constを使って変数宣言すべきである

## プリミティブ型

- 文字列・数値・真偽値・BigInt・null・undefined・シンボル
- 数値は整数小数どちらも含み、IEEE 754倍精度浮動小数点数
- BigIntはES2020から
	- 任意精度の整数を表す
	- BigIntリテラルは `123n`のようにnを書く
- 文字列型
	- ダブルクオートとシングルクオート
		- 好みの問題だが、統一すべき

真偽値への型変換の規則

|型|真偽値への変換結果|
|---|---|
|数値|0とNaNがfalse, それ以外true|
|BigInt|0nがfalse, それ以外true|
|文字列|空文字列("")だけがfalse, それ以外true|
|null, undefined| falseになる|
|オブジェクト|すべてtrue|

## 演算子

- NaNには比較演算子が通じない
	- オペランドにNaNが与えられた時は常にfalse
- 短絡評価ある
- 
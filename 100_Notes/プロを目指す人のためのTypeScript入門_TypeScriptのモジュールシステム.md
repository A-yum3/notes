## import宣言とexport宣言

### 変数のエクスポートとインポート

モジュール内のトップレベルでの変数宣言の前にexportと書くことによって、その変数はエクスポートされるようになる

```ts
export const name = "uhyo";
export const age = 26;
```

```ts
import { name, age } from "./uhyo.js";

console.log(name, age);
```

```ts
import { uhyoName, age as uhyoAge } from "./uhyo.js";
```

### 関数もエクスポートできる

```ts
export const getUhyoName = () => {
	return "uhyo";
};

export function getUhyoName() {
	return "uhyo";
};
```

クラス宣言もできる

### defaultエクスポートとdefaultインポート

```ts
let value = 0;

export default function increment() {
	return ++value;
}
```

```ts
import increment from "./counter.js";
```

defaultエクスポートというのは**暗黙のうちに変数を用意し、defaultという名前でエクスポートする**

```ts
const uhyoAge = 26;

export {
	uhyoAge as default
}
```

- defaultエクスポートは基本的に使用しない？
	- エディタによるサポートが弱いから

### 型のインポート・エクスポート

```ts
export type Anima = {
	species: string;
	age: number;
}
```

`export type {}`という構文もあり、型としてのみ使用可能となる。

### その他の関連構文

`import * as 変数名 from モジュール名;`という形式。モジュール名前空間オブジェクトが代入される

## DefinitelyTypedと@types

npmで配布されているモジュールにはTypeScript向けの型定義が同梱されているものがあり、これをサポートするために運用されているのがDefinitelyTypedおよび@typesパッケージ

### @typesパッケージのインストール

### 自分で型定義ファイルを作るには

- `.d.ts`ファイルに書くのが普通
- 
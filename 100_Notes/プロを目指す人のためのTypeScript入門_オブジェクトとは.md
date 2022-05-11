## オブジェクトとは

- 連想配列である
- スプレッド構文
- 参照コピーはそのまま代入
- 値渡しはスプレッド構文を使うと実現する（スプレッドは新しいオブジェクトを作る）

## オブジェクトの型

- typeで型に名前をつける
- interface宣言でオブジェクト型を宣言する
	- ほとんどの場合type文で代用可能
	- 大体type使ってれば良い
- インデックスシグネチャ
	- どんな名前のプロパティも受け入れる
	- TypeScriptが保証する型安全性を破壊する可能性がある
	- 型安全性のためには避けるべき
	- Mapオブジェクトのが安全なのでそっちを使う
- オプショナルなプロパティの宣言
- 読み取り専用プロパティの宣言
- typeofキーワードで変数の型を得る
	- 型を作るキーワード
- typeof演算子と誤解しないようにする
	- 式を作る
- typeof（型の方）は濫用すべきでない

## 部分型関係

- TypeScriptの型システムの根幹

### 部分型とは

```typescript
type FooBar = {  
    foo: string;  
    bar: number;  
}  
  
type FooBarBaz = {  
    foo: string;  
    bar: number;  
    baz: boolean;  
}  
  
const obj: FooBarBaz = {  
    foo: "hi",  
    bar: 1,  
    baz: false  
};  
  
const obj2: FooBar = obj;
```

- 2つの型の互換性を表す概念
- TypeScriptにおける部分型関係は**構造的部分型**
	- よく見られるのは**名前的部分型**

### プロパティの包含関係による部分型関係の発生

- プロパティの包含関係によって部分型関係を説明できる

2つの条件が満たされればSがTの部分型となる
1. Tが持つプロパティはすべてSにも存在する
2. 条件1の各プロパティについて、Sにおけるそのプロパティの型はTにおけるプロパティの型の部分型（または同じ型）である

```typescript
// T
type Animal = {  
    age: number;  
};  

// S
type Human = {  
    age: number;  
    name: string;  
};
```

```typescript
type AnimalFamily = {  
    familyName: string;  
    mother: Animal;  
    father: Animal;  
    child: Animal;  
}  
  
type HumanFamily = {  
    familyName: string;  
    mother: Human;  
    father: Human;  
    child: Human;  
}
```

- 基本的には部分型関係は「ある型をほかの方の代わりに使える」
- 「ある方をほかの型と見なせる」という直感的な概念に基づく

## 型引数を持つ型

- 型引数は、型を定義するときにパラメータを持たせることができる

### 型引数を持つ型を宣言する

```typescript
type User<T> = {  
    name: string;  
    child: T;  
}
```

型引数を持つ型は**ジェネリック型**とも呼ばれる

### 型引数を持つ型を使用する

```ts
const obj: Family<number, string> = {  
    mother: 0,  
    father: 100,  
    child: "1000"  
};
```

- Familyの定義は具体的な型には言及せずに「構造」にのみ言及している
- ある種の抽象化であり、それゆえに高度に抽象化されたプログラムを書くためには型引数を持つ型が不可欠

### 部分型関係による型引数の制約

- `extends`構文を使用できる
- 型引数の宣言の後ろに`extends`型を付加できる
	- この型引数は常に型の部分型でなければいけない、という制約を意味する

```ts
type HasName = {  
    name: string;  
};  
  
type Family<Parent extends HasName, Child extends HasName> = {  
    mother: Parent;  
    father: Parent;  
    Child: Child;  
};
```

```ts
type Animal = {  
    name: string;  
};  
  
type Human = {  
    name: string;  
    age: number;  
};  
  
type HasName = {  
    name: string;  
};  
  
type Family<Parent extends HasName, Child extends HasName> = {  
    mother: Parent;  
    father: Parent;  
    Child: Child;  
};  
  
// OK  
type S = Family<Animal, Human>;  
  
// Error(AnimalはHumanの部分型ではない)
type T = Family<Human, Animal>;
```

### オプショナルな型引数

```ts
type Animal = {  
    name: string;  
}  
  
type Family<Parent = Animal, Child = Animal> = {  
    mother: Parent;  
    father: Parent;  
    child: Child;  
}
```

## 配列

- 配列はオブジェクトの一種

### 配列型の記法

```ts
const arr1: boolean[] = [false, true];  
  
const arr2: Array<{  
    name: string  
}> = [  
    {name: "山田さん"},  
    {name: "田中さん"},  
    {name: "鈴木さん"}  
]
```


### readonly配列型

```ts
const arr: readonly number[] = [1, 10, 100];
```

### 配列の機能

- pushメソッド
	- 末尾に追加
- lengthプロパティ
- MDN参照
	- JavaScriptの知識をTypeScriptに生かす

### for-of文によるループ

```ts
const arr = [1, 10, 100];  
  
for (const elm of arr) {  
    console.log(elm);  
}
```

### イテレータ

- `[Symbol.iterator]`メソッドを用いてイテレータ取得可能

### タプル型

```ts
let tuple: [string, number] = ["foo", 0];  
tuple = ["aiueo", -555];
```

### 配列の要素アクセス時の危険性

- 存在しないプロパティにアクセスするとTypeScriptはundefinedを返す
- インデックスアクセスは極力使用しない

## 分割代入

### オブジェクトの分割代入-基本パターン

```ts
const { foo, bar } = obj;
```

- 分割代入で宣言された変数には型注釈がつけられない

### オブジェクトの分割代入-ネストしたパターン

```ts
const nested = {  
    num: 123,  
    obj: {  
        foo: "hello",  
        bar: "world"  
    }  
}
```

- 実用されるのはせいぜい２、３段階程度

### 配列の分割代入

```ts
const arr = [ 1, 2, 4, 8, 16];  
  
  
const [, foo, bar, , baz] = arr;
```

### 分割代入のデフォルト値

```ts
type Obj = { foo?: number };  
const obj1: Obj = {};  
const obj2: Obj = {foo: -1234};  
  
// fooに500が代入  
const {foo = 500} = obj1;  
  
/// barに-1234が入る  
const {foo: bar = 500} = obj2;
```

- **デフォルト値はundefinedのみに対して適用される**

### restパターンでオブジェクトの残りを取得する

```ts
const { foo, ...restObj } = obj;
```

```ts
const obj = {  
    foo: 123,  
    bar: "string",  
    baz: false,  
};  
  
const {foo, ...restObj} = obj;  
  
console.log(foo);  
console.log(restObj);
```

## その他の組み込みオブジェクト

### Dateオブジェクト

- 日時を表す組み込み
- 使いにくいオブジェクト
	- ミュータブルである
	- タイムゾーン周りの扱いが難しい
- Temporalという新しい組み込みオブジェクトが進んでいる

### 正規表現オブジェクト

- replaceとmatch
- 名前付きキャプチャリンググループ
	- (?<グループ名>)

### Mapオブジェクト・Setオブジェクト

```ts
const map: Map<string, number> = new Map();  
map.set("foo", 1234);  
  
console.log(map.get("foo"));  
console.log(map.get("bar"));
```

### プリミティブなのにプロパティがある

- 文字列と数値
	- str.length
- BigInt

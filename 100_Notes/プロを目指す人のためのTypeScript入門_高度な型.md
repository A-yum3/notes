## ユニオン型とインターセクション型

### ユニオン型の基本

- 「または」「合併型」

```ts
type Animal = {  
    species: string;  
}  
  
type Human = {  
    name: string;  
}  
  
type User = Animal | Human;  
  
const tama: User = {  
    species: "Felis silvestris catus"  
}  
  
const uhyo: User = {  
    name: "uhyo"  
};
```

### 伝搬するユニオン型

```ts
type Animal = {  
    species: string;  
    age: string;  
}  
  
type Human = {  
    name: string;  
    age: number;  
}  
  
type User = Animal | Human;  
  
const tama: User = {  
    species: "Felis silvestris catus",  
    age: "永遠の17歳"  
}  
  
const uhyp: User = {  
    name: "uhyo",  
    age: 26  
};
```

ageは `string | number`型になる

### インターセクション型とは

- 交差型
- オブジェクト型を拡張した新しい型を作る

```ts
type Animal = {  
    species: string;  
    age: number;  
}  
  
type Human = Animal & {  
    name: string;  
}  
  
const tama: Animal = {  
    species: "Felis silvestris catus",  
    age: 3  
}  
  
const uhyo: Human = {  
    species: "Homo sapiens sapiens",  
    age: 26,  
    name: "uhyo"  
}
```

### ユニオン型とインターセクション型の表裏一体な関係

- 反変の位置

### オプショナルプロパティ再訪

```ts
type Human = {  
    name: string;  
    age?: number;  
}  
  
const uhyo: Human = {  
    name: "uhyo",  
    age: 25  
};  
  
const john: Human = {  
    name: "John Smith",  
};
```

意味合い的には同じだが明示的に書くとこう

```ts
type Human = {  
    name: string;  
    age: number | undefined;  
}  
```

- データがないかもしれない可能性を`undefined`で表したいだけであって、省略自体に強い動機がない場合はこっちのほうが適している
	- ageのデータが存在する可能性と存在しない可能性があることを型で表現しつつ、書き忘れた際にコンパイルエラーとすることができる
- `exactOptionalPropertyTypes`が `age?: number`のようなオプショナルプロパティに明示的にundefinedを代入することができなくなる

### オプショナルチェイニングによるプロパティアクセス

- アクセスされるオブジェクトがnullやundefinedでも使用できる

## リテラル型

- 重要

### 4種類のリテラル型

- プリミティブ型をさらに細分化した型

```ts
const foo: "foo" = "foo";  
const one: 1 = 1;  
const t: true = true;  
const three: 3n = 3n;
```

### テンプレートリテラル型

```ts
function getHelloStr(): `Hello, ${string}!` {  
    const rand = Math.random();  
    if (rand < 0.3) {  
        return "Hello, world!";  
    } else if (rand < 0.6) {  
        return "Hello, my world!";  
    } else if (rand < 0.9) {  
        return "Hello, world."; // error  
    } else {  
        return "Hell, world!"; // error  
    }  
}
```

### ユニオン型とリテラル型を組み合わせて使うケース

- ユニオン型が鍵
- リテラル型のユニオン型が頻出パターン

```ts
function signNumber(type: "plus" | "minus") {  
    return type === "plus" ? 1 : -1;  
}
```

### リテラル型のwidening

- リテラル型が自動的に対応するプリミティブ型に変化するという挙動

```ts
const uhyo1 = "uhyo";  
let uhyo2 = "uhyo";
```

- letの場合は、変数の型がリテラル型に推論されそうな場合はプリミティブ型に変換する
- オブジェクトリテラルの型が推論されるとき、各プロパティの型がリテラル型となる場合はwideningされる
	- あとから書き換え可能だから
- 書き換えたい場合は多くなく、その場合はreadonlyプロパティを持つ型の型注釈を明示的に書く

### wideningされるリテラル型・wideningされないリテラル型

- wideningされるリテラル型は**式としてのリテラルに対して型推論で推論されたもの**

```ts
const uhyo1 = "uhyo"; // widening  
const uhyo2: "uhyo" = "uhyo"; // wideningされない  
  
let uhyo3 = uhyo1;  
let uhyo4 = uhyo2;
```

必要に応じて型注釈を書くことで全て解決してしまう

## 型の絞り込み

- 型の絞り込みはコントロールフロー解析と呼ばれることもある

### 等価演算子を用いる絞り込み

```ts
type SignType = "plus" | "minus";  
  
function signNumber(type: SignType) {  
    return type === "plus" ? 1 : -1;  
}  
  
function numberWithSign(num: number, type: SignType | "none") {  
    if (type === "none") {  
        return 0;  
    } else {  
        return num * signNumber(type);  
    }  
}
```

### typeof演算子を用いる絞り込み

```ts
function formatNumberOrString(value: string | number) {  
    if (typeof value === "number") {  
        return value.toFixed(3);  
    } else {  
        return value;  
    }  
}
```

### 代数的データ型をユニオン型で再現するテクニック

- 代数的データ型
	- タグ付きユニオン・直和型といった別名もある
	- TypeScriptにはない
- **扱うデータの形と可能性を型で正確に表現する**

```ts
type Animal = {  
    tag: "animal";  
    species: string;  
}  
type Human = {  
    tag: "human";  
    name: string;  
}  
type User = Animal | Human;
```

- tagプロパティが与えられている

```ts
type Animal = {  
    tag: "animal";  
    species: string;  
}  
type Human = {  
    tag: "human";  
    name: string;  
}  
type User = Animal | Human;  
  
function getUserName(user: User) {  
    if (user.tag === "human") {  
        return user.name;  
    } else {  
        return "名無し";  
    }  
}  
  
const tama: User = {  
    tag: "animal",  
    species: "Felis silvestris catus"  
};  
  
const uhyo: User = {  
    tag: "human",  
    name: "uhyo"  
};
```

- 型のリフレクションが不可能なので、tagを付与して対応している
- typeという名前でも良い

### switch文でも型を絞り込める

```ts
type Animal = {  
    tag: "animal";  
    species: string;  
}  
type Human = {  
    tag: "human";  
    name: string;  
}  
type User = Animal | Human;  
  
function getUserName(user: User): string {  
    switch(user.tag) {  
        case "human":  
            return user.name;  
        case "animal":  
            return "名無し";  
    }  
}
```

## keyof型・lookup型

### lookup型とは

```ts
type Human = {  
    type: "human";  
    name: string;  
    age: number;  
};  
  
function setAge(human: Human, age: Human["age"]) {  
    return {  
        ...human,  
        age  
    };  
}  
  
const uhyo: Human = {  
    type: "human",  
    name: "uhyo",  
    age: 26  
};  
  
const uhyo2 = setAge(uhyo, 27);
```

- 型情報を再利用する
- 使いすぎは良くない（一見して具体的な部分がわからない）

### keyof型とは

- オブジェクト型からそのオブジェクトのプロパティ名の型を得る機能

```ts
type Human = {  
    name: string;  
    age: number;  
};  
  
type HumanKeys = keyof Human;  
  
let key: HumanKeys = "name";  
key = "age";
```

```ts
const mmConversionTable = {  
    mm: 1,  
    m: 1e3,  
    km: 1e6,  
};  
  
function coverUnits(value: number, unit: keyof typeof mmConversionTable) {  
    const mmValue = value * mmConversionTable[unit];  
    return {  
        mm: mmValue,  
        m: mmValue / 1e3,  
        km: mmValue / 1e6  
    };  
}
```


- keyof typeof 変数 のパターン

### keyof型・lookup型とジェネリクス

keyofは型変数と組み合わせて使うことができる

```ts
function get<T, K extends keyof T>(obj: T, key: K): T[K] {  
    return obj[key];  
}  
  
type Human = {  
    name: string;  
    age: number;  
}  
  
const uhyo: Human = {  
    name: "uhyo",  
    age: 26  
};  
  
const uhyoName = get(uhyo, "name");  
const uhyoAge = get(uhyo, "age");
```

## asによる型アサーション

- **型アサーションの使用はできるだけ避けるべき**
	- TypeScriptが保証してくれる型安全性を意図的に破壊する機能
- 使い所は？
	- TypeScriptの型推論が及ばない時

### 型アサーションを用いて式の型をごまかす

- 型アサーションは**式 as 型**という構文で、その式の型を強制的に変える

正しい使い所は？
-> 不正確な型を正しく直すために使っているとき

```ts
type Animal = {  
    tag: "animal";  
    species: string;  
}  
type Human = {  
    tag: "human";  
    name: string;  
}  
type User = Animal | Human;  
  
function getNamesIfAllHuman(users: readonly User[]): string[] | undefined {  
    if (users.every(user => user.tag === "human")) {  
        return (users as Human[]).map(user => user.name);  
    }  
    return undefined;  
}
```

### as constの用法

- 構文は `式 as const`
- 4つの効果を及ぼす
	- 配列リテラルの型推論結果を配列型ではなくタプル型にする
	- オブジェクトリテラルから推論されるオブジェクト型はすべてのプロパティがreadonlyになる。配列リテラルから推論されるタプル型もreadonlyタプル型になる
	- 文字列・数値・BigInt・真偽値リテラルに対してつけられるリテラル型がwideningしないリテラル型になる
	- テンプレート文字列リテラルの型がstringではなくテンプレートリテラル型になる


```ts
const names1 = ["uhyo", "John", "Taro"]; // string[]型  
const names2 = ["uhyo", "John", "Taro"] as const; // readonly ["uhyo", "John", "Taro"]型
```

## any型とunknown型

- 最凶の危険性を誇る機能・any型

### any型という最終兵器

- **型チェックを無効化する型**
- any型に対してはTypeScriptによる型チェックがまったく行われない

### any型の存在理由

- 移行しやすくするため
- any型が型をうまく表現出来ない場合のエスケープハッチとしての機能を持つ
- anyを使う前にasやユーザー定義型ガードで「正しい状態をTypeScriptに教える」

### anyに近いが安全なunknown型

- **なんでも入れられる型**
- どうやって使えばよいのか？
	- 型の絞り込みを行う

```ts
function useUnknown(val: unknown) {  
    if (typeof val === "string") {  
        // string  
    } else {  
        // not string  
    }  
}
```

## さらに高度な型

### object型・never型

- object型
	- プリミティブ以外すべて
- never型
	- 当てはまる値が存在しない

### 型述語（ユーザー定義型ガード）

- anyやasの仲間で、型安全性を破壊する恐れのある危険な機能

```ts
function isStringOrNumber(value: unknown): value is string | number {  
    return typeof value === "string" || typeof value === "number";  
}  
  
const something: unknown = 123;  
  
if (isStringOrNumber(something)) {  
    console.log(something.toString());  
}
```

- 引数名 is 型　という構文

```ts
type Human = {  
    type: "Human";  
    name: string;  
    age: number;  
};  
  
function isHuman(value: any): value is Human {  
    if (value == null) return false;  
    return (  
        value.type == "Human" &&  
        typeof value.name === "string" &&  
        typeof value.age === "number"  
    );  
}
```

- asserts 引数名 is 型　という形もある

```ts
type Human = {  
	type: Humna;
    name: strin;  
    age: number;  
};  
  
function assertHuman(value: any): asserts value is Human {  
    if (value == null) {  
        throw new Error("Given value is null or undefined");  
    }  
    if (  
        value.type !== "Human" ||  
        typeof value.name !== "string" ||  
        typeof value.age !== "number"  
    ) {  
        throw new Error('Given value is not a Human');  
    }  
}
```

### 可変長タプル型

- タプル型の亜種
- ...配列型を含んだ形

```ts
type NumberAndStrings = [number, ...string[]];  
  
const arr1: NumberAndStrings = [25, "uhyo", "hyo", "hyo"];  
const arr2: NumberAndStrings = [25];
```

- ...配列型はタプル型の中で1回しか使えない制限がある

### mapped types

```ts
type Fruit = "apple" | "orange" | "strawberry";  
  
type FruitArrays = {  
    [P in Fruit]: P[]  
};  
  
const numbers: FruitArrays = {  
    apple: ["apple", "apple"],  
    orange: ["orange", "orange", "orange"],  
    strawberry: []  
};
```

### conditional types

型の条件分岐を行うための型

```ts
type RestArgs<M> = M extends "string" ? [string, string] : [number, number, number];  
  
function func<M extends "string" | "number">(  
    mode: M,  
    ...args: RestArgs<M>  
) {  
    console.log(mode, ...args);  
}

func("string", "uhyo", "hyo");
func("number", 1, 2, 3);
```

- union distribution（ユニオンの分配）という性質がある

### 組み込みの型を使いこなす

`Readonly<T>`

Tに与えられたオブジェクト型のすべてのプロパティを読み取り専用にしたオブジェクト型

```ts
type T = Readonly<{  
    name: string;  
    age: number;  
}>;
```

`Partial<T>`

Tのすべてのプロパティをオプショナルにした型

`Required<T>`

Tのすべてのプロパティをオプショナルではなくした型

```ts
type T = Partial<{
	name: string;
	age: number;
}>
```

`Pick<T, K>`

Tというオブジェクト型のうちKで指定した名前のプロパティのみ残したオブジェクト型

`Omit<T, K>`

Kで指定されたプロパティを除いたオブジェクト型

```ts
type T = Pick<{
	name: string;
	age: number;
}, "age">;
```

`Extract<T, U>`

T（普通はユニオン型）の構成要素のうちUの部分型であるもののみを抜き出した新しいユニオン型

`Exclude<T, U>`

Tの構成要素のうちUの部分型であるもののみを取り除いた新しいユニオン型

```ts
type Union = "uhyo" | "hyo" | 1 | 2 | 3;  
type T = Extract<Union, string>;
```

`NonNullable<T>`

Exclude<T, null | undefined> と同様で、Tからnullとundefinedの可能性を除いた型

### anyをisで書き換えてみる

```ts
type Human = {  
    type: "Human";  
    name: string;  
    age: number;  
};  
  
function isPropertyAccessible(value: unknown): value is { [key: string]: unknown } {  
    return value != null;  
}  
  
function isHuman(value: unknown): value is Human {  
    if (!isPropertyAccessible(value)) return false;  
    return (  
        value.type === "Human" &&  
        typeof value.name === "string" &&  
        typeof value.age === "number"  
    );  
}
```
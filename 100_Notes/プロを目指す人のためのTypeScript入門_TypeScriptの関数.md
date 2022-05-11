## 関数の作り方

### 関数宣言で関数を作る

```ts
function range(min: number, max: number): number[] {  
    const result = [];  
    for (let i = min; i <= max; i++) {  
        result.push(i);  
    }  
    return result;  
}  
  
console.log(range(5, 10));
```

- 巻き上げがある

### 関数式で関数を作る

```ts
type Human = {  
    height: number;  
    weight: number;  
}  
  
const calcBMI = function (human: Human): number {  
    return human.weight / human.height ** 2;  
};  
  
const uhyo: Human = {height: 1.84, weight: 72};  
console.log(calcBMI(uhyo));
```

- 関数も値の一種である

### アロー関数式で関数を作る

```ts
type Human = {  
    height: number;  
    weight: number;  
};  
const calcBMI = ({  
    height, weight  
}: Human):number => {  
    return weight / height ** 2;  
};  
const uhyo: Human = { height: 1.84, weight: 72};  
console.log(calcBMI(uhyo));
```

### アロー関数式の省略形

```ts
const calcBMI = ({  
    height, weight  
}: Human): number => weight / height ** 2;
```

### メソッド記法で関数を作る

```ts
const obj = {  
    double(num: number): number {  
        return num * 2;  
    },  
    double2: (num: number): number => num * 2,  
};
```

### 可変長引数の宣言

- rest引数構文を使って実現できる
- 配列型orタプル型が必要

```ts
const sum = (...args: number[]): number => {  
    let result = 0;  
    for (const num of args) {  
        result += num;  
    }  
    return result;  
}
```

### 関数呼びだしでのスプレッド構文

```ts
const sum = (...args: number[]): number => {  
    let result = 0;  
    for (const num of args) {  
        result += num;  
    }  
    return result;  
}  
const nums = [1,2,3,4,5];  
console.log(sum(...nums));
```

### オプショナル引数の宣言

```ts
const toLowerOrUpper = (str: string, upper?: boolean): string => {  
    if (upper) {  
        return str.toUpperCase();  
    } else {  
        return str.toLowerCase();  
    }  
}
```

### コールバック関数を使ってみる

- 関数の引数として関数を渡す

```ts
type User = { name: string; age: number };  
const getName = (u: User): string => u.name;  
const users: User[] = [  
    { name: "uhyo", age: 26 },  
    { name: "John Smith", age: 15 }  
];  
  
const names = users.map(getName);  
console.log(names);
```

## 関数の型

### 関数型の記法

```ts
type F = (repeatNum: number) => string;  
  
const xRepeat: F = (num: number): string => "x".repeat(num);
```

- 関数型の引数に名前が付いているのは、エディタを充実させるため

### 返り値の型注釈は省略可能

- 省略すると型推論される

### 返り値の型注釈は省略すべきか

- 明示する理由
	- 関数内部で返り値の型に対して型チェックを働かせられる
- 基本的には返り値の型を明記した方が有利

### 引数の型注釈が省略可能な場合

```ts
const xRepeat = (arg: number): string => "x".repeat(arg);
```

- contextual typing

```ts
type F = (arg: number) => string;  
const xRepeat: F = (num) => "x".repeat(num);
```

### コールシグネチャによる関数型の表現

```ts
type MyFunc = {  
    isUsed?: boolean;  
    (arg: number): void  
};  
  
const double: MyFunc = (arg: number) => {  
    console.log(arg * 2)  
}  
  
double.isUsed = true;  
console.log(double.isUsed);  
  
double(1000);
```

## 関数型の部分型関係

### 返り値の型による部分型関係

(引数リスト) => Sという関数型は(引数リスト) => Tという関数型の部分型である

```ts
type HasName = {  
    name: string;  
}  
  
type HasNameAndAge = {  
    name: string;  
    age: number;  
}  
  
const fromAge = (age: number): HasNameAndAge => ({  
   name: "John Smith",  
   age,  
});  
  
const f: (age: number) => HasName = fromAge;  
const obj: HasName = f(100);
```

### 引数の型による部分型関係

- 「Tを引数に受け取る関数」が「Sを引数に受け取る関数」の部分型である
- HasNameAndAge型はHasName型の部分型
	- 「HasNameを引数に受け取る関数」の型は「HasNameAndAgeを引数に受け取る関数」の型の部分型になるということ

```ts
type HasName = {  
    name: string;  
}  
  
type HasNameAndAge = {  
    name: string;  
    age: number;  
}  
  
const showName = (obj: HasName) => {  
    console.log(obj.name);  
};  
const g: (obj: HasNameAndAge) => void = showName;  
  
g({  
    name: "uhyo",  
    age: 26  
});
```

- 関数型の返り値の値の型は関数型の**共変**
	- 順方向の部分型関係
- 関数型の引数の型は**反変**
	- 逆方向の部分型関係

### 引数の数による部分型関係

```ts
type UnaryFunc = (arg: number) => number;  
type BinaryFunc = (left: number, right: number) => number;  
const double : UnaryFunc = arg => arg * 2;  
const add: BinaryFunc = (left, right) => left + right;  
  
const bin: BinaryFunc = double;  
console.log(10, 100); // 20
```

## ジェネリクス

ジェネリクスは、型引数を受け取る関数を作る機能

### 関数の型引数とは

```ts
function repeat<T>(element: T, length: number): T[] {  
    const result: T[] = [];  
    for (let i = 0; i < length; i++) {  
        result.push(element);  
    }  
    return result;  
}  
  
console.log(repeat(<string>"a", 5));  
console.log(repeat<number>(123, 3));
```

- 関数<型引数たち>(引数たち)という形をとる
- 入力値がどんな値でもいいな、と思った場合はジェネリクスが使える可能性が高い

### 関数の型引数を宣言する方法

関数の名前がないとき

```ts
const repeat = function<T>(element: T, length: number): T[] {  
    const result: T[] = [];  
    for (let i = 0; i < length; i++) {  
        result.push(element);  
    }  
    return result;  
}
```

アロー関数の場合
```ts
const repeat = <T>(element: T, length: number): T[] => {  
    const result: T[] = [];  
    for (let i = 0; i < length; i++) {  
        result.push(element);  
    }  
    return result;  
}
```

### 関数の型引数は省略できる

明示しなかった場合、型推論によって補われる

```ts
function repeat<T>(element: T, length: number): T[] {  
    const result: T[] = [];  
    for (let i = 0; i < length; i++) {  
        result.push(element);  
    }  
    return result;  
}  
  
const result = repeat("a", 5);
```

### 型引数を持つ関数型

型引数を取る関数を引数に受け取る高階関数のようなものを定義したいときには出番になる

```ts
type Func = <T>(arg: T, num: number) => T[];  
  
const repeat: Func = (element,length) => {  
    const result = [];  
    for (let i = 0; i < length; i++) {  
        result.push(element);  
    }  
    return result;  
}
```

## 変数スコープと関数

- 関数スコープ
- ブロックスコープ

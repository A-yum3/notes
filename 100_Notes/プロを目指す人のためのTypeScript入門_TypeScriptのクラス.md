## クラスの宣言と使用

### クラス宣言とnew構文

- プロパティ宣言
- オプショナル

```ts
class User {  
    name: string = "";  
    age?: number;  
}  
  
const uhyo = new User();
```

### メソッド宣言

```ts
class User {  
    name: string = "";  
    age: number = 0;  
  
    isAdult(): boolean {  
        return this.age >= 20;  
    }  
  
    setAge(newAge: number) {  
        this.age = newAge;  
    }  
}  
  
const uhyo = new User();
```

### コンストラクタ

```ts
class User {  
    name: string;  
    age: number;  
  
    constructor(name: string, age: number) {  
        this.name = name;  
        this.age = age;  
    }  
  
    isAdult(): boolean {  
        return this.age >= 20;  
    }  
}  
  
const uhyo = new User("uhyo", 26);
```

- staticプロパティとstaticメソッドもあるよ
- public, protected, privateの3種類があるよ

php8っぽく書けるよ

```ts
class User {  
    constructor(public name: string, private age: number) {}
}
```

### クラス式でクラスを作成する

```ts
const user = class {  
    name: string;  
    age: number;  
  
    constructor(name: string, age: number) {  
        this.name = name;  
        this.age = age;  
    }  
  
    public isAdult(): boolean {  
        return this.age >= 20;  
    }  
}
```

### もう一つのプライベートプロパティ

```ts
const user = class {  
    name: string;  
    #age: number;  
  
    constructor(name: string, age: number) {  
        this.name = name;  
        this.#age = age;  
    }  
  
    public isAdult(): boolean {  
        return this.#age >= 20;  
    }  
}
```

迷ったら#の方がJavaScriptにもある機能なので厳格

### クラスの静的初期化ブロック

```ts
class User {  
    static adminUser: User;  
    static {  
        this.adminUser = new User();  
        this.adminUser.#age = 9999;  
    }  
  
    #age: number = 0;  
    getAge() {  
        return this.#age;  
    }  
    setAge(age: number) {  
        if (age < 0 || age > 150) {  
            return;  
        }  
        this.#age = age;  
    }  
}
```

### 型引数を持つクラス

```ts
class User<T> {  
    name: string;  
    #age: number;  
    readonly data: T;  
      
    constructor(name: string, age: number, data: T) {  
        this.name = name;  
        this.#age = age;  
        this.data = data;  
    }  
    public isAdult(): boolean {  
        return this.#age >= 20;  
    }  
}
```

## クラスの型

### クラス宣言はインスタンスの型を作る

- クラス宣言した場合は型宣言できる
	- クラス式で行った場合は型は作られない

### newシグネチャによるインスタンス化可能性の表現

new (引数リスト) => インスタンスの型

```ts
class User {  
    name: string = "";  
    age: number = 0;  
}  
  
type MyUserConstructor = new () => User;  
  
const MyUser: MyUserConstructor = User;  
const u = new MyUser();  
console.log(u.name, u.age);
```

### instanceof演算子と型の絞り込み

- インスタンスとはあくまでnewされたオブジェクトを指す

```ts
class User {  
    name: string = "";  
    age: number = 0;  
}  
  
const uhyo = new User();  
console.log(uhyo instanceof User); //true  
console.log({} instanceof User); //false  
  
const john: User = {  
    name: "John Smith",  
    age: 15,  
};  
  
console.log(john instanceof User); //false
```

## クラスの継承

- extendsで継承ができる
- overrideもある
- super呼び出しもできる

### override修飾子とその威力

```ts
class User {  
    name: string;  
    #age: number;  
  
    constructor(name: string, age: number) {  
        this.name = name;  
        this.#age = age;  
    }  
  
    public isAdult(): boolean {  
        return this.#age　>= 20;  
    }  
}  
  
class PremiumUser extends User {  
    rank: number = 1;  
      
    public override isAdult(): boolean {  
        return true;  
    }  
}
```

- 明示的にoverrideできる
- `noImplicitOverride => true`コンパイラオプションと組み合わせることで効果を発揮する

### privateとprotectedの動作と使い所

- 他言語と代替一緒
- privateプロパティは親と子で同じ名前のプライベートプロパティを定義できないが、#ならできる

### implementsキーワードによるクラスの型チェック

- クラス名 implements 型 {...}
- 宣言の意図を明確化したいときは積極的にimplements

## this

### 関数の中のthisは呼び出し方によって決まる

- thisが具体的に何を指すかは関数の呼び出し方によって決まる

### アロー関数におけるthis

- thisを外側の関数から受け継ぐ
- アロー関数は自分自身のthisを持たない
- 基本的にアロー関数を使う

### thisを操作するメソッド

- メタプログラミング気味
- applyメソッド
	- func.apply(obj, args)の形で呼び出すことで、「関数funcを、中でのthisをobjにして呼び出す」という意味になる
	- func.bind(obj)は返り値として「呼び出し時のthisがobjに固定されたfunc関数」が得られる

### 関数の中以外のthis

- トップレベルではundefined

### 例外処理

- 例外をキャッチするtry-catch文
- 失敗を表すthrow以外の選択肢は、失敗を表す値を返す
	- 型情報の面で優れている
- try-catch文は大域脱出という特徴を活かせる
	- 複数箇所で発生した例外を１ヵ所で処理できる
- finallyで脱出に割り込む
-  throwは何でも投げられる


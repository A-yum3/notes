## チェックの厳しさに関わるオプション

### チェックをまとめて有効にできるstrictオプション


strictオプションで以下が有効になる

- noImplicitAny
- noImplicitThis
- alwaysStrict
- strictBindCallApply
- strictNullChecks
- strictFunctionTypes
- strictPropertyInitialization
- useUnknownInCatchVariables

### strictNullChecksでnullとundefinedを安全に検査する

onかoffで型推論の結果が変わる

```ts
type MaybeHuman = {  
    name?: string;  
}  
  
function func(obj: MaybeHuman) {  
    const name = obj.name;  
    console.log(name);  
}
```

### 型の書き忘れや推論の失敗を防ぐnoImplicitAnyオプション

- 関数宣言における引数の型宣言の取り扱い

### インデックスアクセスを厳しくするnoUncheckedIndexedAccessオプション

- 厳しすぎる

### 新規プロジェクトでのおすすめ設定

- なるべく厳しい設定にする
- strictオプションは有効にする
- noUncheckedIndexedAccessも有効にする
- exactOptionalPropertyTypesも有効にする
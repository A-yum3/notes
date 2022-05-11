## Promiseを使う

### コールバック関数の登録とエラー処理

- Promiseオブジェクトに対してはthenメソッドでコールバック関数を登録する

### 自分でPromiseオブジェクトを作る

```ts
const p = new Promise<number>((resolve) => {  
    setTimeout(() => {  
        resolve(100);  
    }, 3000);  
});  
  
p.then((num) => {  
    console.log(`結果は${num}`);  
});
```
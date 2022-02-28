#php/pest 

https://pestphp.com/docs/expectations
Jestを使った人なら馴染みがあると思います。

`expect($value)`に検査したい$valueをセットし、あとはチェーンで検証したいことを連ねます。

### カスタム Expectations
Expectationsを組み合わせて作るオリジナルExpectations

`expect()->extend()`で拡張できる。
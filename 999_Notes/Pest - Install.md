#php/laravel/test #php/pest

https://pestphp.com/docs/installation

composerにインストール
```zsh
composer require pestphp/pest-plugin-laravel --dev
```

tests/Pest.phpを作成
このファイルは最初に呼ばれるファイルになるため、ここにトレイトやクラスを再帰的にバインドできる。
```zsh
php artisan pest:install
```

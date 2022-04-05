#php/laravel/test
ORMとQuery BuilderはConnectionが違う

```php
// テストメソッド内
User::factory()->count(5)->create();
User::whereName('test')->delete();
```

本番コード
```php
DB::table('users')->select(*)->where('name', '=', 'test')->get();
```

テストの中ではnameが`test`のものは消えていて欲しいが、Query BuilderとORMではコネクションが違うため、Deleteはトランザクションがコミットされておらず、適用されない。
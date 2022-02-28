- 誕生日(特殊な日時フォーマットでソート)

```php

public function scopeOrderByBirthday($query)
{
	$query->orderByRaw('date_format(birth_date, "$m-$d")');
}

```

これでもいけるけど、もう少し早くすることができる
- 複合インデックスを作る
- 複数の呼び出し元でレコードを並べるような状況に設定する

```php
public function up()
{
	Schema::create('users', function (Blueprint $table) {
		// ry
		$table->rawIndex("(date_format(birth_date, '%m-%d')), name", "users_birthday_name_index");
	});
}

```

- 今週誕生日のものを抽出する


```php
public function scopeWhereBirthdayThisWeek($query)
{
	$query->whereRaw('date_format(birth_date, "%m-%d") between ? and ?', [
		Carbon::now()->startOfWeek()->format('m-d'),
		Carbon::now()->endOfWeek()->format('m-d'),
	]);
}
```
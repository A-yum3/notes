- カスタムアルゴリズムを使用してレコードをソートする

```php
public function index()
{
	$features = Feature::query()
		->withCount('comments', 'votes')
		->when(request('sort'), function ($query, $sort) {
			switch ($sort) {
				case 'title': return $query->orderBy('title', request('direction'));
				case 'status': return $query->orderByStatus(request('direction'));
				case 'activity': return $query->orderByActivity(request('direction'));
			}
		})
		->latest()
		->paginate();
}
```

- Statusによるソート

```php
public function scopeoderByStatus($query, $direction)
{
	$query->orderBy(DB::raw('
		case
			when status = "Requested" then 1
			when status = "Approved" then 2
			when status = "Completed" then 3
		end
	'), $direction);
}
```

- indexの付け方を工夫する

```php
public function up()
{
	Schema::create('features', function (Blueprint $table) {
		$table->bigIncrements('id');
		$table->string('title', 100);
		$table->string('status', 10);
		$table->timestamps();
		$table->rawIndex('(
			case
				when status = "Requested" then 1
				when status = "Approved" then 2
				when status = "Completed" then 3
			end
		)', 'features_status_ranking_index');
	});
}
```

- カスタムランキングのインデックスは作成不可能(2つのクエリに依存している)
- 
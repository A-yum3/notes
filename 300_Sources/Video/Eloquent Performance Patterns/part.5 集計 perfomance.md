- count()するときにcase文を入れるクエリを発行したい
- `toBase()`
	- データベースからデータを取得するが、Eloquentモデルは準備しない

```php
Model::toBase()
	// case statements
	->selectRaw("count(case when status = 'Requested' then 1 end) as requested")
	// filterを使う(postgres)
	->selectRaw("count(*) filter (where status = 'Completed') as completed")
```
	
上記のように指定することでクエリを1つにまとめることが出来る

---

- `toBase()`
- SQL-> case
- SQL -> filter

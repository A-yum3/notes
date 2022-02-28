- Eloquentの`load`は親データを先に取ってしまった場合にEager Loadingしたい時に使える
- https://qiita.com/shosho/items/abf6423283f761703d01

```php
$feature->load('commnets.user');
$feature->comments->each->setRelation('feature', $feature);
```

setRelationを使うことで動的にRelationを設定できる

---

- `load()`
- `setRelation()`
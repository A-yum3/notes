#php/laravel/test

当然ながら`RefreshDatabase`はDB（プロセス外依存）になるため、動作が遅くなります。
あとできればリポジトリ触る系のコードはアプリケーション層に閉じ込めて、ドメイン層ではモデルだけいじる感じの分離ができたら理想かなぁと考えています。
`RefreshDatabase`は`protected $seeder = SomeTableSeeder::class;`すればシーディングもしてくれる。
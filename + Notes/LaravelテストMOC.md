#MOC #php/laravel/test  

基本方針
[[プライベートメソッドに対してテストは書かない]]
[[publicなAPIに書く]]
[[ドメインのテスト優先]]
[[命名規則は動作内容を付ける]]
[[AAAパターン]]
[[モックについて]]
[[テストメソッドは1つの動作検証につき1つ]]
[[デトロイト派とロンドン派]]
[[全体的な方針はDocで書く]]

Factory

[[Factoryはid決め打ちしない]]
[[複数Factoryはヘルパーに分ける]]
[[LaravelのFactoryベストプラクティス(仮)]]
[[Custom Factory]]
[[Seederの使い道]]

dataProvider

[[dataProviderの小技]]
[[dataProviderはテストメソッドの上]]

ベストプラクティス

アンチパターン
[[テストにおけるif文はアンチパターン]]

注意点

[[markTestInComplete]]
[[ORMとQuery BuilderはConnectionが違う？]]
[[RefreshDatabaseは遅い]]
[[RefreshDatabase Or DatabaseTransactions]]
[[時間が関係するテスト]]
[[assertEqualよりもAssertSame]]

小技
[[JsonResponseを検証する方法]]
[[setUpの使用方法]]



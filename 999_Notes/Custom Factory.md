#php/laravel/test 

Laravelのテストコードを書く時に、MySQLのColumnとしてJson型だったり、リクエストボディでJson形式を使うことは多々ある。(実際には連想配列)

その時にお手製のCustom Factoryを作って生成をコード化する。それによって変更が出た際にも、1点だけ修正すれば良い。

---

### Json Factory(Immutable Class)

ModelにはLaravelがFactoryを用意してくれているが、Json形式のFactoryはないので作成する。
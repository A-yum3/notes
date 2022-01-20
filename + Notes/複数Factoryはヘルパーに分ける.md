#php/laravel/test

Userデータも必要だし、Bookデータも必要だし、User_Bookも必要だし、Historyも必要だし・・・みたいに複数のデータが必要になるとFactoryだけでもテストメソッドが膨張します。  
テストメソッドが膨張すると可読性が落ちるので、保守性も落ちます。  

なのでprivateで別にfactoryするメソッドを用意してしまいましょう。その関数で引数を受け取るようにすればdataProviderで動的にfactoryもできるのでベスト

--- 

- DataSetsディレクトリを作ってDataSetsクラスを作っても良いかもしれない。
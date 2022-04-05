#php/laravel/test 

setTimeNow()は各テストメソッド内で行い、dataProviderで入れ替え可能にすると便利かも
- 普遍的なデータはデータセット専用クラスを用意し、それを簡単に利用するようにする方針案
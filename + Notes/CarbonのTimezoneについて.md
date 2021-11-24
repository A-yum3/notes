
shiftTimezone(tz) タイムゾーンを切り替える（差分計算は行わない）  
timezone(tz) タイムゾーンを切り替える（差分計算を行う）

Model::factory()->create($data)で日付の値を指定した時はアプリの設定関係なしで**指定した時間**がDBに登録される

逆にアプリ内で保存処理をする際はLaravelのapp.phpに書かれているtimezoneに変換されてDBに登録される
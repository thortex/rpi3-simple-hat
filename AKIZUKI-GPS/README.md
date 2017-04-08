# 秋月電子通商製 GPS の接続方法

秋月電子通商より発売されている みちびき対応 GPS を
取り付ける方法を解説します（簡単です）。

## 部品 

必要な部品は二つだけです。本体とコネクターです。

* http://akizukidenshi.com/catalog/g/gK-09991/
* http://akizukidenshi.com/catalog/g/gC-09734/

## 接続方法

本体側は赤枠のピンに青、緑、黄、橙、赤、茶の順番で接続します。

[本体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/Pi-GPIO-header-26-sm.png "本体側")

RPi1b, RPi2, RPi3, RPi0, RPi0W で共通のピン配置のはずです。
RPi1 の Rev. 1 だけは Pin 4 に +5V が来ていないかもしれません。

GPS 側は 5V, GND, RxD, TxD, 1PPS に青、緑、黄、橙、赤、茶の順番で接続します。
茶のピンは使用（接続）しません。

[全体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/overview.jpg "全体")

## ソフトウェア

UART (serial) を raspi-config で有効にする必要があります。
ただし、コンソールログインは使用しません。

また、Bluetooth 機能がついている RPi3, RPi0W は Bluetooth を無効化するか、
コアクロックを 250 MHz に固定化する必要があります。

GPS については既存の他サイトを参考にして頂ければと思います。

     $ sudo apt-get install gpsd gpsd-clients

等として gpsmon とすれば、GPS の捕捉情報が取得できると思います。

単純には sudo head /dev/serial0 | strings とすれば、NMEA データが生で表示できます。

gpsd の場合は /etc/default/gpsd ファイルの DEVICE に /dev/serial0 を指定します。

RPi シリーズは /dev/serial0 に UART (Serial)、/dev/serial1 に Bluetooth が割り当てられるようになっています。






    




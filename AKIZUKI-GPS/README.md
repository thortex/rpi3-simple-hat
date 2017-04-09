# 秋月電子通商製 GPS の接続方法

秋月電子通商より発売されている みちびき対応 GPS を
取り付ける方法を解説します（簡単です）。

## 必要な部材

必要な部材は本体とケーブルの二つだけです。

* [本体](http://akizukidenshi.com/catalog/g/gK-09991/)
* [ケーブル](http://akizukidenshi.com/catalog/g/gC-09731/)

## 初期設定

* ※ GPSを接続する前に行います。
* UART (serail) を有効にする必要があります。
* ※ OnChip Bluetooth の有無により、設定が異なります。

### OnChip Bluetooth 未搭載モデル

UART (serial) を raspi-config で有効にする必要があります。

    $ sudo raspi-config

「5 Interfacing Options」を選択します。

![設定1](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config1.png)

「P6 Serial」を選択します。

![設定2](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config2.png)

「Would you like a login shell to be acessible over serial?」と聞かれますが、
コンソールログインは使用しませんので「いいえ」を選択します。
「はい」を選択してしまうと、GPS側にコンソール（起動ログ等の）文字列が出力
されてしまい、誤動作する可能性があります。

![設定3](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config3.png)

「Would you like to serial port hardware to be enabled?」と聞かれますので、
「はい」を選択します。

![設定4](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config4.png)

確認画面が表示されるので、「了解」を選択します。

![設定5](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config5.png)

「Finish」を選択し、raspi-config を終了します。

![設定6](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/raspi-config6.png)

再起動を促されますが、一度電源を切り、GPS を接続します。

     $ sudo poweroff


### OnChip Bluetooth 搭載モデル (RPi3, RPi0W)

OnChip Bluetooth がついている RPi3, RPi0W は Bluetooth に PL011 UART
が、Serial に mini UART がデフォルトで割り当てられており、
CPU動作周波数変動に伴うコアクロック変動の影響で、mini UART の
ボーレートが一定になりません。これでは通信が安定しないため、
ペリフェラルのコアクロックを 250 MHz に固定化する必要があります。

RPi には 2 種類の UART ペリフェラルが存在します。
一つは低機能版の mini UART、もう一方は高機能版の PL011 UART です。

dtoverlay 機能により Bluetooth に PL011 UART を割り当てるか、Serial 
に PL011 UART を割り当てるかを選択する事ができます。

単純に Bluetooth を無効化し、PL011 UART を Serial に割り当てるには
以下の設定を /boot/config.txt に追加します。

     dtoverlay=pi3-disable-bt

Bluetooth に mini UART、Serial に PL011 UART を割り当てる場合は、
以下の設定を /boot/config.txt に追加します。

     dtoverlay=pi3-miniuart-bt

Bluetooth を無効にすると不便な場合もあると思いますので、
Serial に mini UART、Bluetooth に PL011 UART を割り当て、コアクロックを
250 MHz に固定化する場合、以下の設定を /boot/config.txt に追加します。

     core_freq=250

あとは「OnChip Bluetooth 未搭載モデル」の場合と同様に、raspi-config 
で Serial UART 機能を有効化します。


TODO ここから

## 接続方法

本体側は赤枠のピンに青、緑、黄、橙、赤、茶の順番で接続します。

![本体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/Pi-GPIO-header-26-sm.png "本体側")

RPi1b, RPi2, RPi3, RPi0, RPi0W で共通のピン配置のはずです。
RPi1 の Rev. 1 だけは Pin 4 に +5V が来ていないかもしれません。

GPS 側は 5V, GND, RxD, TxD, 1PPS に青、緑、黄、橙、赤、茶の順番で接続します。
茶のピンは使用（接続）しません。

![全体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/overview.jpg "全体")


GPS については既存の他サイトを参考にして頂ければと思います。

     $ sudo apt-get install gpsd gpsd-clients

等として gpsmon とすれば、GPS の捕捉情報が取得できると思います。

単純には sudo head /dev/serial0 | strings とすれば、NMEA データが生で表示できます。

gpsd の場合は /etc/default/gpsd ファイルの DEVICE に /dev/serial0 を指定します。

RPi シリーズは /dev/serial0 に UART (Serial)、/dev/serial1 に Bluetooth が割り当てられるようになっています。





    




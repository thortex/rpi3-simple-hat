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

### 前提条件

Kernel は 4.4.x 〜 4.9.x 程度と仮定します。
パッケージ類は最新であると仮定します。

     $ sudo apt-get update
     $ sudo apt-get upgrade
     $ sudo rpi-update
     

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


## 接続方法

本体側は赤枠のピンに青、緑、黄、橙、赤、茶の順番で接続します。

![本体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/Pi-GPIO-header-26-sm.png "本体側")

RPi1b, RPi2, RPi3, RPi0, RPi0W で共通のピン配置のはずです。
RPi1 の Rev. 1 だけは Pin 4 に +5V が来ていないかもしれません。

GPS 側は 5V, GND, RxD, TxD, 1PPS に青、緑、黄、橙、赤、茶の順番で接続します。
茶のピンは使用（接続）しません。

![全体](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/overview.jpg "全体")

## 動作確認

/dev/serial0 を head して、$GPGGA や GPGSA 等で始まる行が表示されていれば、
正常に動作していると判断できます。


     $ sudo head /dev/serial0 | strings 

下記は位置決めに必要な衛星が捕捉出来ていない状態のログです。

     $GPGGA,122708.288,,,,,0,3,,,M,,M,,*47
     $GPGSA,A,1,,,,,,,,,,,,,,,*1E
     $GPRMC,122708.288,V,,,,,0.07,143.30,090417,,,N*48
     $GPZDA,122708.288,09,04,2017,,*53
     $GPGGA,122708.288,,,,,0,3,,,M,,M,,*47
     (?$GPGSA,A,1,,,,,,,,,,,,,,,*1E

### gpsd + gpsmon による確認

gpsd をインストールすると、Serial 経由の GPS 情報を解析し、localhost:2947
で GPS 位置情報を取得できるようになります。apt-get で gpsd と clients を
インストールします。

     $ sudo apt-get install gpsd gpsd-clients


等として gpsmon とすれば、GPS の捕捉情報が取得できると思います。

/dev/serial0 を入力デバイスとして設定するため、 /etc/default/gpsd ファイル
の DEVICES という行を以下のように編集します。

     DEVICES="/dev/serial0"

編集後、gpsd を再起動し、起動状態を確認します。

     $ sudo systemctl restart gpsd.service
     $ sudo systemctl status gpsd.service

以下のように Loaded が loaded、Active が active (running) になって
いれば正常に起動しています。

     ● gpsd.service - GPS (Global Positioning System) Daemon
        Loaded: loaded (/lib/systemd/system/gpsd.service; static)
        Active: active (running) since 日 2017-04-09 20:22:32 JST; 1h 12min ago
      Main PID: 824 (gpsd)
        CGroup: /system.slice/gpsd.service
                └─824 /usr/sbin/gpsd -N /dev/serial0


正常起動を確認したら、 gpsmon コマンドを起動し、衛星が捕捉できているかを
確認します。

     $ gpsmon

起動ログ例を以下に示します。


![gpsmon](https://raw.githubusercontent.com/thortex/rpi3-simple-hat/master/AKIZUKI-GPS/images/gpsmon.png "")

捕捉衛星数が三つだと正確な位置情報が取得できません。
四つ以上捕捉出来ている状態で正確な位置情報が取得出来ていると思われます。

捕捉衛星数は 右中央の「GGA」にある「Sats」という項目の値です。

条件が良ければ屋内でも 4 衛星を捕捉可能だと思います。


他にも有益なコマンドがいくつかあります。

* xgps - X 上で gpsd の動作確認する簡易クライアント。
* xgpsspeed - GPS 情報による速度計。
* gegps - Google Earth に位置情報を送る。
* gpsctl - GPS の設定。
* gpspipe - gpsd の出力情報をパイプする。
* gpxlogger - トラッキング情報の保存。
* lcdgps - 4x40 LCD に表示する。

### 1 PPS の使用方法

Kernel の再コンパイルをしている例も他のウェブサイトでありますが、
少なくとも 4.4.x 以降の Kernel であれば、1PPS に対応しています。

/boot/config.txt に以下の行を追加します。

     dtoverlay=pps-gpio,gpiopin=18

信号の立ち下がりで PPS 信号を検知したい場合は、assert_falling_edge
を指定します。デフォルトは信号の立ち上がり検知です。

### NTP との連携

T.B.D (chronyd, ntpd)

# Navit

Navit はフリーの地図表示アプリケーションです。
GPS との連携も可能です。ルート案内も可能です。

## インストール

apt-get で navit と maptool を指定します。

     sudo apt-get install maptool navit

## 地図データのダウンロード

地図は OpenStreetMap データが使用可能です。
http://download.geofabrik.de/asia/japan.html のサイトから japan-latest.osm.bz2
をダウンロードし、maptool で navit 用ファイルに変換する方法があります。

     $ mkdir ~/navit
     $ cd ~/navit
     $ wget -c http://download.geofabrik.de/asia/japan-latest.osm.bz2
     $ bzcat japan-latest.osm.bz2 | maptool -S 100000000 japan.bin

変換には長い時間がかかります。すでに navit 用に変換済みのファイルを
ダウンロードする事も可能です。

http://maps3.navit-project.org/ のサイトで
「Predefined Area」→「Asia」→「Japan+Korea+Taiwan」を選択し、
「Get map!」ボタンをクリックする事で navit 用の地図ファイルを
ダウンロード可能です。

## フォント設定

デフォルトの状態だとフォントは Liberation Sans が選択されていますので、
日本語フォントに変更します。    

    $ cp /etc/navit/navit.xml ~/.navit/
    $ cd ~/.navit/
    $ sed -i TBD

TBD 

## 起動














    




# IoT program by MicroPython

ESP32 開発ボードを利用して、様々なセンサの値をサーバに送信するためのプログラム群です。送信方法は MQTT になります。

## 構成

```
micropython-iot
    +-- app.py # 実行ファイル
    +-- config.py # 環境別設定ファイル（config.default.py をリネーム）
    +-- main.py # ボードが実行するファイル
    +-- ulib # ライブラリ群
        +-- cds.py # 光センサのライブラリ
        +-- umqtt.py # MQTT通信のライブラリ
```

環境ごとの設定は `config.py` で行います。  
`app.py` の `main` メソッドにボードで動かしたいプログラムを記述します。

## MicroPython インストール

使用するボードは `ESP32` を想定しています。手順は公式ドキュメントのとおりです。

ESP32 での MicroPython の始め方  
https://micropython-docs-ja.readthedocs.io/ja/latest/esp32/tutorial/intro.html

### 仮想環境

Python の仮想環境を作成します。

```
% python3 -V
Python 3.9.1
% python3 -m venv venv
% source venv/bin/activate
```

#### 仮想環境を抜ける場合

```
(venv) % deactivate
```

### 必要なユーティリティ

ボードを扱うために必要なユーティリティをインストールします。

**esptool**

ESP8266 または ESP32 のROMブートローダと通信するためのユーティリティ  
https://github.com/espressif/esptool

```
pip install esptool
```

**ampy**

シリアル接続を介して CircuitPython または MicroPython ボードと対話するためのユーティリティ  
https://github.com/scientifichackers/ampy

```
pip install adafruit-ampy
```

### ファームウェアのダウンロード

ESP32用のファームウェア  
https://micropython.org/download/esp32/

最新の安定版をダウンロードします。

> GENERIC : esp32-idf4-20210202-v1.14.bin

### ボードの確認

接続しているボードを確認します。

```
ls -al /dev/tty.*
・・・
crw-rw-rw-  1 root  wheel   18,   4  3 23 13:48 /dev/tty.SLAB_USBtoUART
・・・
```

表示されたシリアルポートは以降の操作で使用しますので控えておいてください。

### ボードの初期化

次のコマンドを実行して、ボードのフラッシュを消去します。

```
esptool.py --port /dev/tty.SLAB_USBtoUART erase_flash
```

### ファームウェアの書き込み

ダウンロードしたファームウェアを指定してボードに書き込みます。

```
esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART write_flash 0x1000 esp32-idf4-20210202-v1.14.bin
```

#### コマンドオプションについて

- `--chip` : ボード（チップ）の種類
- `--port` : ボードが接続されたシリアルポート

`0x1000` はコマンドに渡される数値の指定です。（16進数）

### 動作確認

REPLプロンプトを利用して実行します。

```
screen /dev/tty.SLAB_USBtoUART 115200
```

`help()` と入力して enter を押すと次のように表示されます。

```
Welcome to MicroPython on the ESP32!

For generic online docs please visit http://docs.micropython.org/

For access to the hardware use the 'machine' module:
```

`ctrl + a + k` で終了確認のプロンプトが表示されるので `y` を入力して終了します。

## MicroPython の実行

ESP32では、起動時に `boot.py` が実行され、その後存在すれば `main.py` を実行します。自身のプログラムは `main.py` を作成して転送することになります。

### ファイルの転送

`ampy` ユーティリティを使ってファイルの転送を行います。コマンドは次のとおりです。

- get:   ボードからパソコンにファイルを転送する
- ls:    ボード内の一覧を表示する
- mkdir: ボード上でディレクトリを作成する
- put:   パソコンからボードにファイルを転送する
- reset: ボード上でソフトリセットとブートを実行する
- rm:    ボード上でファイルを削除する
- rmdir: ボード上でディレクトリを削除する
- run:   ボード上でプログラムファイルを実行する

例：転送済みのファイルを確認する

```
ampy -p /dev/tty.SLAB_USBtoUART ls
/boot.py
```

#### ディレクトリを転送する場合

ディレクトリの転送は、ボード上にディレクトリを作成したあと、パスを指定してファイルを転送します。

例：ulib ディレクトリを転送する

```
ampy -p /dev/tty.SLAB_USBtoUART mkdir ulib
ampy -p /dev/tty.SLAB_USBtoUART put ulib/umqtt.py ulib/umqtt.py
```

### サンプルコードの作成

Lチカのコードを作成してボード上で実行してみます。

```main.py
import machine
import time

p13 = machine.Pin(13, machine.Pin.OUT)

while True:
  p13.on()
  time.sleep(1)
  p13.off()
  time.sleep(1)
```

つづいて `main.py` を転送します。

```
ampy -p /dev/tty.SLAB_USBtoUART put main.py
```

D13ピンにLEDを接続すると、1秒間隔で点滅します。

## プログラムの使い方

このプログラム群では、`main.py` の中で `app.py` を実行しています。使い方のながれは次のとおりです。

1. micropython-iot 直下の config.default.py をコピーして config.py を作成
1. 環境に合わせて config.py を修正して転送
1. micropython-iot 直下の app.py を修正して転送
1. ulib 内の必要なファイルを転送
1. micropython-iot 直下の main.py を転送

転送コマンドの例

```
ampy -p /dev/tty.SLAB_USBtoUART put config.py
ampy -p /dev/tty.SLAB_USBtoUART put app.py
ampy -p /dev/tty.SLAB_USBtoUART mkdir ulib
ampy -p /dev/tty.SLAB_USBtoUART put ulib/umqtt.py ulib/umqtt.py
ampy -p /dev/tty.SLAB_USBtoUART put ulib/cds.py ulib/cds.py
ampy -p /dev/tty.SLAB_USBtoUART put main.py
```

ディレクトリは最初の転送時のみ作成します。以降は作成する必要はありません。

### プログラムを止めたい場合

ボード上の main.py を削除することでプログラムが止まります。

```
ampy -p /dev/tty.SLAB_USBtoUART rm main.py
```

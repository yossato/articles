# raspi‐jpn‐char‐keyboard で日本語入力対応 USB HID キーボードを作成！

2025/02/08  

こんにちは！  
今回は Raspberry Pi Zero W を使って、ホストPCに日本語（漢字を含む）の入力を送信できる USB HID キーボードを実現するプロジェクト「raspi‐jpn‐char‐keyboard」をご紹介します。  
Githubで公開しています。  

[https://github.com/yossato/raspi-jpn-char-keyboard](https://github.com/yossato/raspi-jpn-char-keyboard)

USB Gadget モードと Windows の IME を利用することで、Raspberry Pi Zero W が自動的に HID キーボードとして振る舞う仕組みを構築します。

<iframe width="560" height="315" src="https://www.youtube.com/embed/QMJmQvIDZQU?si=z_7HtEQnojOCUS-b" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>  
  
元ネタは@nanbuwksさんのQiitaの記事の中にある [MS-IME の場合は 文字コード(ex,611b・・・「愛」の Unocode)+F5でコード入力できるっぽいですね](https://qiita.com/nanbuwks/items/a791f99ff7773d1313b6#windows-の場合) です。  

---

## 目次

1. [必要な準備](#必要な準備)
2. [USB Gadget モードの有効化](#usb-gadget-モードの有効化)
   - [config.txt の編集](#configtxt-の編集)
   - [cmdline.txt の編集](#cmdlinetxt-の編集)
3. [USB ポート接続の注意点](#usb-ポート接続の注意点)
4. [リポジトリのクローンとサービス設定](#リポジトリのクローンとサービス設定)
5. [動作確認：キー入力の送信](#動作確認キー入力の送信)
7. [まとめ](#まとめ)

---

## 必要な準備

1. **OS イメージの書き込み**  
   Raspberry Pi Imager を使用して、Raspberry Pi OS のイメージを microSD カードに書き込みます。
   <img width="792" alt="Raspberry Pi" src="https://github.com/user-attachments/assets/25a1fb75-0c50-4c83-8c02-c1176d26aa91" />

   書き込み後、SD カードは PC に差し込んで直接ファイル編集が可能です。

   > **Tip:** Raspberry Pi Imager の高度な設定機能を使えば、Wi-Fi の SSID/パスワードや SSH の有効化、ユーザー名・パスワードの設定も事前に行えます。  
   > これにより、初回起動時にモニターやキーボードを接続せずに、すぐにリモートから SSH でアクセス可能になります。

3. **必要な設定ファイルの編集準備**  
   - **config.txt**  
   - **cmdline.txt**

---

## USB Gadget モードの有効化

Raspberry Pi Zero W を USB HID キーボードとして動作させるためには、システム起動時に `dwc2` モジュールをロードし、USB Gadget の設定を有効にする必要があります。以下の 2 つのファイルを適切に編集してください。

### 1. cmdline.txt の編集

**/boot/firmware/cmdline.txt** の内容には、すでに `modules-load=dwc2` が含まれていることを確認します。  
自分の環境の場合、動作確認が取れた時の内容は次のようになっています:

```txt
console=serial0,115200 console=tty1 root=PARTUUID=503e009b-02 rootfstype=ext4 fsck.repair=yes rootwait modules-load=dwc2 quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=JP
```

この設定により、起動時に自動的に `dwc2` モジュールがロードされ、USB Gadget モードが利用可能になります。  
**※ cmdline.txt は 1 行で記述されているため、改行や不要な空白を入れないように注意してください。**

### 2. config.txt の編集

**/boot/firmware/config.txt** では、USB Gadget 用のオーバーレイ設定として `dtoverlay=dwc2` を追加する必要があります。ただし、追加する位置が非常に重要です。

具体的には、`max_framebuffers=2` の直後に追記してください。  
以下は、編集例です:

```ini
# Enable DRM VC4 V3D driver
dtoverlay=vc4-kms-v3d
max_framebuffers=2

dtoverlay=dwc2

# Don't have the firmware create an initial video= setting in cmdline.txt.
# Use the kernel's default instead.
disable_fw_kms_setup=1

# Disable compensation for displays with overscan
disable_overscan=1

# Run as fast as firmware / board allows
arm_boost=1
```

**ポイント:**

- **正しい位置に追記することが必須！**  
  `dtoverlay=dwc2` は `max_framebuffers=2` の**直後の行**に記述してください。  
  これにより、USB Gadget モード（HID キーボード機能）が確実に有効になります。

- この設定後、Raspberry Pi Zero W は USB 接続時に HID デバイス（例：/dev/hidg0）として認識され、ホストPCへのキーストローク送信が可能となります。

上記の編集を行った後、SD カードを Raspberry Pi に戻して起動すれば、USB Gadget モードが正しく機能するはずです。  

---

## USB ポート接続の注意点

Raspberry Pi Zero W には、**USB** ポートと **PWR IN** ポートという 2 つの microUSB ポートがあります。  
**ホストPCと接続する際は、必ず「USB」と刻印されたポートを使用してください。**

- **正しい接続例:**  
  ホストPCの USB ケーブルは「USB」ポートに接続し、USB Gadget モードとして動作させる。  

- **誤った接続例:**  
  「PWR IN」ポートに接続してしまうと、電源供給専用になり、ガジェットモードの設定が反映されず、ホスト側で入力が認識されなくなります。

![USB ポートの接続イメージ](https://github.com/user-attachments/assets/1eabff18-5d8e-476f-8076-2c2d66e69716)


詳しくは、[こちらのQiita記事](https://qiita.com/zaburo/items/ea185925a46877a6ad26)も参考にしてください。

---

## リポジトリのクローンとサービス設定

プロジェクトの全設定はリポジトリ内のスクリプトで管理されています。  
以下の手順でクローンおよびセットアップを行ってください。

1. **リポジトリのクローン**

   ```bash
   git clone git@github.com:yossato/raspi-jpn-char-keyboard.git
   ```

2. **サービス用スクリプトの確認と実行**  
   リポジトリ内には、次の重要なスクリプトが含まれています:

   - `setup_usb_hid_gadget_service.sh`  
     → systemd サービスファイルを作成し、USB HID Gadget モードの自動起動を設定します。  
     ※実行権限がない場合は、`chmod +x setup_usb_hid_gadget_service.sh` を実行してください。

   - `setup_usb_hid_gadget.sh`  
     → 実際の USB HID キーボードとしての各種設定を行います。

   - `cleanup_usb_hid_gadget_service.sh`  
     → 設定を解除するためのクリーンアップスクリプトです。

3. **サービスの有効化**

   以下のコマンドを sudo 権限で実行して、サービスを登録・起動します。

   ```bash
   sudo ./setup_usb_hid_gadget_service.sh
   ```

   設定が成功すると、システム起動時に自動的に USB HID キーボードのサービスが開始されるようになります。

---

## 動作確認：キー入力の送信

リポジトリ内の `send_key_via_hid.py` を使って、実際にホストPCにキー入力を送信してみましょう。  
テスト用のテキストファイル（例：`sample.txt`）を用意し、次のコマンドを実行します。

```bash
python3 send_key_via_hid.py -d /dev/hidg0 -t sample.txt
```

*注意:*  
- `/dev/hidg0` は USB Gadget モードで生成されるデバイスファイルです。  
- デバイスファイルへの読み書き権限はデフォルトで666に設定されるようにsystemdサービスの中で毎回設定していますが、セキュリティーを考慮する場合は適宜アクセス権を設定してください。

---

## まとめ

今回のプロジェクト「raspi‐jpn‐char‐keyboard」では、以下のポイントが重要です。

- **OS イメージの書き込み前に、config.txt と cmdline.txt の正しい編集を実施すること。**
- **ホストPCとの接続は、必ず「USB」と刻印されたポートを使用すること。**
- **リポジトリ内のスクリプトを活用して、USB HID Gadget モードを自動起動させること。**
- **send_key_via_hid.py を利用して、/dev/hidg0 へキー入力を送信し、ホストPCに入力を反映させること。**

これらの設定を正しく行えば、Raspberry Pi Zero W が日本語入力対応の USB HID キーボードとして動作し、ホストPCへの入力が可能になります。  
ぜひ、あなたの環境でも試してみてください！

---

参考: [Qiitaの記事](https://qiita.com/zaburo/items/ea185925a46877a6ad26)

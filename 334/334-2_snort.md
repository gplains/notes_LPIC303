---
title:334-snort
---
# 334-ネットワーク侵入検知
 
## snort(2.9系)

 - 303-300ではsnort2.9系を想定している

 -  snort3.0系はコンフィグの保存場所を含めほぼ別物

### Ubuntu18.04/20.04の場合

- インストール

  パッケージのインストール中に「監視するNICの指定」を求められる
  ```
  # 予めどのNICが存在するか目を通しておく
  ip a 
  # apt一発
  sudo apt install snort 
  # バージョン確認
  sudo snort -V # 20.04では2.9.7が返る
  ```

- 使用
  
  ```
  # 受信したパケットを/tmp/test.(タイムスタンプ) に格納する
  sudo snort -v -L /tmp/test

  # 採取したトラヒックログは tshark等で解析できる
  sudo tshark -r /tmp/test.(タイムスタンプ)
  ```

### CentOS7の場合

- インストール
  
  https://snort.org/#get-started を参照

  https://www.snort.org/ にアクセス後、 豚マーク → Get Started → CentOS に遷移
  ```
  yum install https://snort.org/downloads/snort/
  yum install https://snort.org/downloads/snort/snort-2.9.18-1.centos8.x86_64.rpm
  ```

### Snortのオプション

### ルールファイル

 - 「コミュニティルール」を使う

### Snort のシグネチャ
  ルールアクション、プロトコル、IPアドレス、ポート番号、方向演算子、IPアドレス、ポート番号
- ルールアクション
- プロトコル 
- IPアドレス
- ポート番号
- 方向演算子
- オプション



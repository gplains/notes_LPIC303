---
title:334-snort
---
# 334-ネットワーク侵入検知
 
## snort(2.9系)

 - 303-300ではsnort2.9系を想定している

 -  snort3.0系はコンフィグの保存場所を含めほぼ別物

### Ubuntu18.04の場合

- インストール
  ```
  sudo apt install snort 
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



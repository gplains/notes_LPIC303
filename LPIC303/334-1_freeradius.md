---
title:freeradius
---
# 334-ネットワークハーデニング

## freeradius

本来であれば 802.1x対応のスイッチとセットで行うべきだが、簡易検証としてfreeradiusをインストールした端末で完結させる

## インストール

- apt インストール

  Ubuntu 20.04 の場合、 3.20 がインストールされるようだ
  ``` 
  sudo apt install freeradius
  ```
- 設定ファイル

  ```
  # 基本フォルダ
  /etc/freeradius/3.0
  # サーバ設定 (基本、変更不要)
  /etc/freeradius/3.0/radiusd.conf
  #クライアント設定 (認証スイッチが増えたときはこちらを変更)
  /etc/freeradius/3.0/clients.conf
  # ユーザ設定  (利用者が増えたときはこちらを変更)
  /etc/freeradius/3.0/users
  ```
- サービス再起動
  ```
  sudo systemctl restart freeradius.service
  ```

## ログインアカウントの追加

認証スイッチに対して、利用者が増えたときはログインアカウントを追加する

- 設定ファイルの修正
  ```
  sudo vi /etc/freeradius/3.0/users
  
  # bob で始まる行の近くに追記
  user Cleartext-Password := "sample"  
  ```
- 接続試行
  ```
  radtest -4 user sample localhost 12345 testing123
  > Received Access-Accept Id 159 from 127.0.0.1:1812 to 127.0.0.1:41817 length 20 が返る
  ```

## 認証可能なスイッチの追加

- 設定ファイルの修正

  ```
  sudo vi /etc/freeradius/3.0/clients.conf
  # コンフィグファイルの末尾に追記
  client somehost {
        ipaddr = 198.51.100.100
        secret = testing123      # 対象の認証スイッチに追記
  }
  ```


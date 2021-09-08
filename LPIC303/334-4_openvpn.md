---
title:334-OpenVPN
---

# 334-VPN


## OpenVPN

- Ubuntuの場合、比較的容易にインストール/設定が可能

## 参考にしたもの

OpenVPN公式の「静的鍵」を参照しています

http://www.openvpn.jp/document/statickey-mini-howto/

### Ubuntu 20.04 の場合

- 今回はリモートアクセスVPNを考える。検証環境の場合、サイト間のパブリックIP同士、サイト１のプライベート側、サイト２のプライベート側で都合3つVLANを切る
  
  サイト１
  | セグメント   | IPアドレス            |
  | ------- | ----------------- |
  | パブリック側  | 198.51.100.100/24 |
  | プライベート側 | 192.168.10.1/24   |
  
  サイト２
  | セグメント   | IPアドレス           |
  | ------- | ---------------- |
  | パブリック側  | 198.51.100.101/24 |

- インストールに際して、別途いくつかのノードを作成する

  ルータノード
  | セグメント | IPアドレス          |
  | ----- | --------------- |
  | サイト１側 | 198.51.100.1/24 |
  | 管理/外抜け| 任意(192.168.81.xx/24) |
 
  サイト１子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.10.100/24       |

- インストール(サイト1/サイト2とも)
  
  ```
  # 予めip_forwardを許可
  sudo vi /etc/sysctl.conf
  > net.ipv4.ip_forward=1
  sudo apt install openvpn
  ```

## 事前共有鍵の作成と配布

- 事前共有鍵の作成

  ```
  openvpn --genkey --secret static.key
  mv secret.key /tmp
  ```
- 対向へ事前共有鍵を配布

  ```
  scp /tmp/static.key user@198.51.100.101:/tmp
  ```

## 設定ファイルの作成

- サーバ側
  
  ```
  sudo cp /tmp/static.key  /etc/openvpn/server/
  # ファイルを編集
  sudo vi /etc/openvpn/server/server.conf
  # 以下ファイル内容
  dev tun
  ifconfig 10.8.0.1 10.8.0.2         # トンネル用IP設定
  secret static.key
  ```

- クライアント側

  ```
  sudo cp /tmp/static.key  /etc/openvpn/client/
  # ファイルを編集
  sudo vi /etc/openvpn/client/client.conf
  # 以下ファイル内容
  remote 198.51.100.100                # サイト1のパブリックIP
  dev tun
  ifconfig 10.8.0.2 10.8.0.1           # サイト1とは対称にする
  route 192.168.10.0 255.255.255.0      # サイト1のプライベートセグメント
  secret static.key
  ```

## 疎通テスト

- サーバ側
  ```
  sudo openvpn --config /etc/openvpn/server/server.conf  \
  --secret /etc/openvpn/server/static.key
  ```
- クライアント側
  ```
  sudo openvpn  --config /etc/openvpn/client/client.conf \
  --secret /etc/openvpn/client/static.key
  ```

この状態でクライアント側でもう一個SSH セッションを張っておいて、
サイト1配下の適当なノードIPにpingを飛ばしておくと、OpenVPNが疎通したタイミングでpingが通るようになる
---
title:334-WireGuard
---

# 334-VPN


## WireGuard

- Ubuntuの場合、比較的容易にインストール/設定が可能

## 参考にしたもの

- gihyoの記事[第614回　WireGuardでVPNサーバーを構築する]

  https://gihyo.jp/admin/serial/01/ubuntu-recipe/0614
- WireGuardでSite to site VPN

  https://blog.asterism.xyz/posts/2020-04-16/

### Ubuntu 20.04 の場合

- 後述のように、NICに設定したIPのほかに「VPN用の仮IP」を設定する必要がある

  今回はサイト間VPNを考える。検証環境の場合、サイト間のパブリックIP同士、サイト１のプライベート側、サイト２のプライベート側で都合3つVLANを切る
  
  サイト１
  | セグメント   | IPアドレス            |
  | ------- | ----------------- |
  | パブリック側  | 198.51.100.100/24 |
  | プライベート側 | 192.168.10.1/24   |
  | VPN仮IP  | 10.0.0.1/30                  |
  
  サイト２
  | セグメント   | IPアドレス           |
  | ------- | ---------------- |
  | パブリック側  | 203.0.113.100/24 |
  | プライベート側 | 192.168.20.1/24  |
  | VPN仮IP  | 10.0.0.2/30                 |

- インストールに際して、別途いくつかのノードを作成する

  ルータノード
  | セグメント | IPアドレス          |
  | ----- | --------------- |
  | サイト１側 | 198.51.100.1/24 |
  | サイト２側 | 203.0.113.1/24                |
  | 管理/外抜け| 任意(192.168.81.xx/24) |
 
  サイト１子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.10.100/24       |

  サイト2子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.20.100/24       |

- インストール(サイト1/サイト2とも)
  
  ```
  # 予めip_forwardを許可
  sudo vi /etc/sysctl.conf
  > net.ipv4.ip_forward=1
  sudo apt install wireguard
  ```

- 秘密鍵/公開鍵の作成(サイト1/サイト2とも)
  
  ```
  wg genkey | sudo tee /etc/wireguard/server.key # 秘密鍵
  sudo cat /etc/wireguard/server.key | wg pubkey | sudo tee /etc/wireguard/server.pub # 公開鍵
  sudo chmod 600 /etc/wireguard/server.pub
  ```

- wireguard設定ファイルの作成(サイト1)
  
  ```
  sudo vi /etc/wireguard/wg0.conf
  [Interface]
  PrivateKey = (サイト1の秘密鍵)
  Address = 10.0.0.1/30  # サイト1のVPN仮IP
  ListenPort = 51820
  PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
  
  [Peer]
  EndPoint= 203.0.113.100:51820 # サイト2のパブリック側IP
  PublicKey =(サイト2の公開鍵)
  AllowedIPs = 10.0.0.2, 192.168.20.0/24 # サイト2のVPN仮IP , サイト2のプライベート側セグメント
  ```

- wireguard設定ファイルの作成(サイト2)
  
  ```
  sudo vi /etc/wireguard/wg0.conf
  [Interface]
  PrivateKey = (サイト2の秘密鍵)
  Address = 10.0.0.2/30  # サイト2のVPN仮IP
  ListenPort = 51820
  PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
  
  [Peer]
  EndPoint= 198.51.100.100:51820 # サイト1のパブリック側IP
  PublicKey =(サイト1の公開鍵)
  AllowedIPs = 10.0.0.1, 192.168.10.0/24 # サイト1のVPN仮IP , サイト1のプライベート側セグメント
  ```

- サービス有効化(サイト1/サイト2とも)

  ```
  sudo systemctl enable wg-quick@wg0
  sudo systemctl start  wg-quick@wg0
  ```

### CentOS8 の場合

- 非推奨(RHEL8ではlibreswanのみサポート)のため省略

## 引っかかりどころ

- IPアドレス

  NICにNetworkManagerで割り付けたIPアドレス(パブリック/プライベート)のほかにVPN用で仮IPを設定する必要がある

- 設定ファイル

  wg0.conf の書き方を誤るとセグメントが全断するので、予め検証環境で試そう

  Address区 / AllowedIPs区 は、最初はVPN仮IP同士だけ書いておいて、``` sudo wg-quick up wg0 ``` が問題なかったら実際に「サービス有効化」するとよい。

  この場合の切り戻しは ```sudo wg-quick down wg0 ``` あるいは潔く仮想マシン再起動

## 今回の検証で困ったこと

- 何かいい作図ツールないかなぁ...

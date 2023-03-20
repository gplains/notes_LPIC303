---
title:Ubuntu20.04でLibreSwanのサンプル
---

# Ubuntu20.04でLibreSwanのサンプル


## 参考にしたもの
- 技術メモの壁

  https://fsck.jp/?p=972
- RHEL8 第4章 IPSEC を使用した VPN の設定

  https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/securing_networks/configuring-a-vpn-with-ipsec_securing-networks#creating-a-host-to-host-vpn_configuring-a-vpn-with-ipsec


## Ubuntu の場合

- 今回はサイト間VPNを考える。検証環境の場合、サイト間のパブリックIP同士、サイト１のプライベート側、サイト２のプライベート側で都合3つVLANを切る
  
  サイト１
  | セグメント   | IPアドレス            |
  | ------- | ----------------- |
  | パブリック側  | 198.51.100.100/24 |
  | プライベート側 | 192.168.10.1/24   |
  
  サイト２
  | セグメント   | IPアドレス           |
  | ------- | ---------------- |
  | パブリック側  | 198.51.100.101/24 |
  | プライベート側 | 192.168.20.1/24  |

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

  サイト2子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.20.100/24       |


- インストール
  
  ```
  sudo apt update ; sudo apt upgrade
  sudo vi /etc/sysctl.conf
  > net.ipv4.ip_forward=1
  sudo apt install libreswan
  ```

- 鍵作成(サイト1/サイト2それぞれで実施)
  ```
  sudo rm /var/lib/ipsec/nss/*.db
  sudo ipsec initnss --nssdir /var/lib/ipsec/nss
  sudo ipsec newhostkey --output /etc/ipsec.d/hostkey.secrets
  > Generated RSA key pair with CKAID "...." was stored in the NSS database
  # ↑生成されたCKAIDに着目
  sudo ipsec showhostkey --left --ckaid "...."   
  # "leftrsasigkey..." を自分側のipsec.conf に添付
  sudo ipsec showhostkey --right --ckaid "...."   
  # "rightrsasigkey..." を相手側のipsec.conf に添付...実はleftsigkeyと一緒
  ```

- 設定ファイルの作成

  サイト1は以下の設定、サイト2はleftとrightを逆に記述する(例:leftidをrightidにする)
  
  片側(VPNを張りに行く側)だけ、``` auto=start ```を追記
  ```
  sudo vi /etc/ipsec.d/site2site.conf
  # 以下
  conn mytunnel
    leftid=198.51.100.100
    left=198.51.100.100
    leftrsasigkey=0sAwEAAaVlhzkYnWm[...]
    leftsubnet=192.168.10.0/24
    rightid=198.51.100.101
    right=198.51.100.101
    rightrsasigkey=0sAwEAAe4Eoycmnk[...]
    rightsubnet=192.168.20.0/24
    authby=rsasig
    # auto=start # 片方だけコメントアウト解除
  ```

- ipsecサービスの再起動
   ```
   sudo systemctl restart ipsec
   sudo  ipsec auto --add mytunnel
   sudo ipsec auto --up mytunnel
   ```

## 引っかかりどころ


- 経路設定(踏み台)

  NW構成は「技術メモの壁」を参考にしたのですが、思ったよりlibreswanの設定ファイル修正がきつかったです。
  今回はルータ(てっぺん)にSSH接続できるようにして、これを踏み台にして各VPNノードにSSH接続できるようにしました。ていうかこんなのVMware Workstation のウィンドウから入力できねえ。

- ipsecコマンドの仕様変更

  一部のコマンド引数が変わってしまった

  ```bash
  # 従前の例
  ipsec newhostkey --output /etc/ipsec.d/hostkey.secrets
  # 実際の例
  ipsec newhostkey
  > Generated RSA key pair with CKAID (...) was stored in the NSS database
  > The public key can be displayed using: ipsec showhostkey --left --ckaid (...)
  ```

- leftとかrightとか

  自分(local)=left 、対抗(remote)=right 。

  さっきの ipsec newhostkeyも、対抗で –left オプションで出力しといてrightrsasigkeyにコピペしとけばよし。逆もまた然り。


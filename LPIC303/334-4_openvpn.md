---
title:334-OpenVPN
---

# 334-VPN


## OpenVPN

- Ubuntuの場合、比較的容易にインストール/設定が可能

## 参考にしたもの

easy-rsa を使用した「一番あたらしい」手順として以下を参考にしました..が、OpenVPN Clientのところは読み飛ばしています。

https://tryota.hatenablog.com/entry/2021/04/11/Ubuntu_Server_%E3%81%A7_OpenVPN%E3%82%B5%E3%83%BC%E3%83%90%E3%82%92%E6%A7%8B%E7%AF%89%E3%81%99%E3%82%8B

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

- インストール(サイト1/サイト2とも)
  
  ```
  # 予めip_forwardを許可
  sudo vi /etc/sysctl.conf
  > net.ipv4.ip_forward=1
  sudo apt install easy-rsa openvpn
  ```

## easy-rsa の設定

```
# 作業フォルダ作成
make-cadir ~/easy-rsa
cd easy-rsa ; ls -l

# パラメタ修正
cp -p vars vars.org
vi vars
# pki初期化
./easyrsa init-pki
# ca作成 パスフレーズの設定を求められる (テストなので0000でよい)
./easyrsa build-ca
# 署名要求の作成
 ./easyrsa gen-req server
# 署名要求から証明書を作成
./easyrsa sign-req server server
# DHパラメタの生成
./easyrsa sign-req server server
# TLS Authキーの生成
 openvpn --genkey --secret ta.key
```

おっと肝心なことを忘れていた。vars の設定内容(のDIFF)

```
diff vars.org  vars
91,96c91,96
< #set_var EASYRSA_REQ_COUNTRY  "US"
< #set_var EASYRSA_REQ_PROVINCE "California"
< #set_var EASYRSA_REQ_CITY     "San Francisco"
< #set_var EASYRSA_REQ_ORG      "Copyleft Certificate Co"
< #set_var EASYRSA_REQ_EMAIL    "me@example.net"
< #set_var EASYRSA_REQ_OU               "My Organizational Unit"
---
> set_var EASYRSA_REQ_COUNTRY   "US"
> set_var EASYRSA_REQ_PROVINCE  "California"
> set_var EASYRSA_REQ_CITY      "San Francisco"
> set_var EASYRSA_REQ_ORG       "Copyleft Certificate Co"
> set_var EASYRSA_REQ_EMAIL     "me@example.net"
> set_var EASYRSA_REQ_OU                "My Organizational Unit"
```

## 生成したファイルをOpenVPNに格納する

```
# ca.crt
sudo cp pki/ca.crt /etc/openvpn/server/
# server.crt
sudo cp pki/issued/server.crt /etc/openvpn/server/
# server.key
sudo cp pki/private/server.key /etc/openvpn/server/
# dh2048.pem
sudo cp pki/dh.pem /etc/openvpn/server/df2048.pem
# ta.key
sudo cp ta.key /etc/openvpn/server/
```

## サーバ側設定ファイルの編集

```
cd /etc/openvpn/server ; pwd
cp -p /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz ./
gzip -d server.conf.gz
# バックアップ採取してから編集
cp -p server.conf server.conf.org
vi server.conf 
# サービス開始
systemctl restart openvpn.service
```

## クライアント秘密鍵/証明書の作成

```
# rootから戻って実行
./easyrsa gen-req client nopass
# クライアント証明書の作成
./easyrsa sign-req client client
```

生成されたファイルを適宜SCPする

一旦、サーバ側からクライアント側の/tmpへscpにて複製

```
# ca.crt
 scp pki/ca.crt user@198.51.100.101:/tmp
# client.key
 scp pki/private/client.key user@198.51.100.101:/tmp
# client.crt
 scp pki/issued/client.crt user@198.51.100.101:/tmp
# ta.key
 scp ta.key user@198.51.100.101:/tmp
```

クライアントにssh接続して適宜移動

```
# ca.crt
sudo mv /tmp/ca.crt /etc/openvpn/client/
# client.crt
sudo mv /tmp/client.crt /etc/openvpn/client/
#client.key
sudo mv /tmp/client.key /etc/openvpn/client/
# ta.key
sudo mv /tmp/ta.key /etc/openvpn/client/
```

## クライアント側設定ファイルの編集

```
# 予め設定ファイルを複製
sudo cp -p /usr/share/doc/openvpn/examples/sample-config-files/client.conf  /etc/openvpn/client/
# バックアップ採取
sudo cp -p /etc/openvpn/client/client.conf /etc/openvpn/client/client.conf.org
 sudo vi /etc/openvpn/client/client.conf
```


## 接続

- サーバ側
  ```
  sudo su -
  openvpn server.conf
  ```
- クライアント側
  ``` 
  sudo su -
  openvpn client.conf
  ```

動作確認は、クライアントに対して別のSSHセッション(あるいはvSphereの仮想コンソール)を開いていくつか試してみる

```
ping 192.168.10.1 # サーバ自身のプライベートIP
ping 192.168.10.100 # サーバ配下のノードIP
```


## 参考:静的鍵

easy-rsa を使わず、OpenVPNビルトインの「静的鍵」を使ったほうが早い(試していないが)

https://www.openvpn.jp/document/statickey-mini-howto/


---
title:Firewalld
---

# Firewalld

厳密にはLPIC303の範囲ではないので(iptablesが範囲なので)

## 設定ファイル

- iptables(参考)
  /etc/sysconfig 配下
- firewalld 
  /etc/firewalld 
  /usr/lib/firewalld

## 基本的なコマンド

- 設定確認
  ```
  sudo firewall-cmd --state 

  sudo systemctl status firewalld
  ```

- ゾーン
  ```
  # ゾーン一覧
  sudo firewall-cmd --get-zones
  sudo firewall-cmd --lista-all-zones
  # internalゾーンの表示
  sudo firewall-cmd --info-zone=internal

  # ゾーンを確認する
  sudo firewall-cmd --get-default-zone
  sudo firewall-cmd --get-active-zones
  
  # デフォルトのゾーンを切り替える
  sudo firewall-cmd --set-default-zone=dmz
  ```
- サービス、ポート
  ```
  # サービスの一覧
  sudo firewall-cmd --get-services
  # ポートの追加
  sudo firewall-cmd --add-port=10051/tcp
  sudo firewall-cmd --add-port={636/tcp,637/tcp,638/udp} --permanent --zone=somezone
  ```
- 設定の反映
  ```
  # permanent で設定したルールを反映する
  sudo firewall-cmd --reload 
  ```

## ZONEの使用の流れ

特定のIP、セグメントだけ許可する場合に使える

```
# 既存のsshサービス待ち受けを削除
sudo firewall-cmd --permanent --remove-service=ssh
# sshzoneを切る
sudo firewall-cmd --permanent --new-zone=sshzone
# sshzoneに接続元IPを追加
sudo firewall-cmd --permanent --zone=sshzone --add-source=192.168.1.2
# sshzoneに待ち受けサービスを追加
sudo firewall-cmd --permanent --zone=sshzone --add-service=ssh
# permanentルールをリロード、上４つの設定がまとめて反映される
sudo firewall-cmd --reload
sudo firewall-cmd --list-all --zone=sshzone
```

## リッチルール

一字一句間違えずに入力が必要...結構大変

使いどころとしては、RHEL8でdeprecatedになってしまった「tcpwrapper」の代替とか
※nftablesでもいいんですが...nftables使うか、firewalld使うかの二択で

```
# 現在のルールを確認
sudo firewall-cmd --list-all

# ルールを追加
sudo firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.1.2" port protocol="tcp" port="23" accept"
# permanentで仕込んだらreloadで反映、すると表示される
sudo firewall-cmd --relaod
sudo firewall-cmd --list-rich-rules --zone=public

# ルールを削除
sudo firewall-cmd --permanent --zone=public --remove-rich-rule="rule family="ipv4" source address="192.168.1.2" port port="23" protocol="tcp" accept" 
# permanentで仕込んだらreloadで反映、すると表示される
sudo firewall-cmd --list-rich-rules --zone=public
sudo firewall-cmd --reload
```

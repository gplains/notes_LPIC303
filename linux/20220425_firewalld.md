---
title:Firewalld
---

# Firewalld

厳密にはLPIC303の範囲ではないので(iptablesが範囲なので)

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

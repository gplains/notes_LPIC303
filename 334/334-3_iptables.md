---
title:334-iptables
---
# 334-パケットフィルタ
 
## iptables
### Ubuntuの場合

- インストール

  基本的にはufwを使う
  18/04からはnftablesが利用可能

  ```
  # iptables-persistent を追加で入れておこう
  sudo apt install iptables-persistent 
  ```

### CentOSの場合

- インストール
  
  7:標準でインストール済み
  8/9:nftablesに移行

### 設定ファイル

- Ubuntu

  /etc/iptables 

- RHEL/CentOS
  
  /etc/sysconfig/iptables
  
```
# 4つのテーブル
FILTER   # 一般的なパケットフィルタ
NAT      # ネットワークアドレス変換
MANGLE   # TOS TTL MARK など
SECURITY # SELinux
```

### コマンド

```
# 現在の設定確認
iptables -L  # IPv4
ip6tables -L # IPv6



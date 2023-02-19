---
title:334-nftables
---
# 334-試験には出ないアプリケーション類
 
## nftables


### Ubuntuの場合

- インストール
  ```
  sudo apt install nftables
  sudo ufw status # ufwが停止していることを確認
  ```

### CentOS8の場合

- インストール
  
  標準でインストール済み

### 設定ファイル


- Ubuntu
  
  /etc/nftables.conf

- RHEL

  /etc/sysconfig/nftables.conf 


```
  # 基本
  flush ruleset # テーブル初期化
  table inet filter # inet テーブルのフィルタ
  chain input # フィルタ本体

  # nftablesのフック
  Prerouting Input Formard Output Postrouting  

  # ルール例
  tcp dport 22 ct stat new accept # 22/tcpを開放
```

### コマンド類

```
# 現在の設定を表示
sudo nft list tables
# 定義済みルールの表示
sudo nft list ruleset 


# 設定ファイルの検証

sufo nft --check -f /etc/nftables.conf

# 設定をファイルからよみこむ
sudo nft -f /etc/nftables.conf 

# inet区が変更されたことを表示
sudo nft list table inet filter
```


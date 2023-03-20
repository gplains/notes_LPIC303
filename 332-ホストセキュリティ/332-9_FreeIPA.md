---
title:332-FreeIPA
---
# 332-FreeIPA

- FreeIPAは v2の試験範囲だがv3では300に移動している

# 構成コンポーネント

- 389 directory server
- MIT kerberos
- Dogtag Certificate System (PKIのweb-ui)
- NTP(chrony) /DNS(BIND) / SSSD
- Certmonger
- IPv6

### Ubuntu18.04の場合

Ubuntu20.04 は　BIND9.16の相性がいまいち

- インストール
  ```
  # クライアント
  sudo apt istal freeipa-client

  # サーバ
  sudo apt install freeipa-server
  ```

### CentOS7の場合

- インストール
  
### 関連コマンド

```
# サーバ導入
ipa-server-install

# レプリカ情報作成
ipa-replica-prepare 

# レプリカ設定取り込み ※伝送はscp/sftpを使う
ipa-replica-install

# レプリカの管理
ipa-replica-manage

# クライアント導入
ipa-client-install


```

- IPAスクリプト
  ```
  # ユーザの作成
  ipa user-add
  # ユーザ削除
  ipa user-del
  # ユーザの検索
  ipa user-find
  # ユーザ情報の表示
  ipa user-show

  # グループの追加
  ipa group-add
  # グループの削除
  ipa group-del
  # グループ情報の表示
  ip group-show 

  # グループにユーザを追加
  ipa grou-add-member (users)

  # 設定オプションの変更
  ipa configh-mod

  # DNSリソースレコードの追加
  ipa dnsrecord-add

  # ドメインの信頼関係の追加
  ipqa trust-add
  # 例:ActiveDirectory とのクロスレルム認証
  ipa trus-add --type=ad (domain) --admin (user) --password
  ```


---
title:332-KERBEROS
---
# 332-ユーザの管理と認証

- ユーザの管理と認証は v2の試験範囲だがv3では300に移動している

## KERBEROS

### Ubuntu 18.04/20.04の場合

- インストール
  ```
  # clientのみ
  sudo apt install krb5-user

  # サーバも
  sudo apt install krb5-kdc krb5-admin-server krb5-config 

  ```

### RHEL/CentOSの場合

- インストール
  ```
  # clientのみ
  sudo dnf install krb5-workstation

  # サーバも
  sudo dnf install krb5-server 
  ```

### 主要コマンド

```
# DB(プリンシパル)の設定
kadmin

# チケット作成
kinit

# チケット表示
klist

# チケット破棄
kdestroy
```
---
title:331-OpenSSL
---

# 331-暗号化

## 331.1 OpenSSL

SSL/TLSの管理を行うツールスタック
秘密鍵、公開鍵の作成及びパラメータの管理
公開鍵の暗号操作
X509証明書、証明書署名要求、および証明書失効リストの作成
証明書署名要求=CSR 、証明書失効リスト=CRL
※OCSP(Online Certificate Status Protocol) : 電子証明書の問い合わせるプロトコル
メッセージダイジェストの計算
RSA暗号方式による暗号化と復号
SSL/TLSクライアントとサーバのテスト
S/MIME署名と暗号化メールの取り扱い
タイムスタンプの要求と生成および検証

OpenSSLの主なサブコマンド

|サブコマンド|説明|
|--|--|
|ciphers|使用可能な暗号スートの一覧表示|
|crl|CRLの管理|
|dgst|メッセージダイジェスト出力|
|genrsa|RSA秘密鍵の生成|
|req|CSRの管理|
|rsa|RSA鍵の管理|
|s_client/s_server|SSL/TLSプロトコルのクライアント/サーバ|
|x509|X509証明書データの管理|


公開鍵/秘密鍵の作成
openssl genrsa –out nopass.key 2048


秘密鍵から公開鍵の作成
openssl rsa –in private.key –pubout –out public.key


CSRの生成
openssl req –new –key 2020.pem –out request.pem


暗号化・署名・認証のためのX.509証明書


Apache HTTPDでのX.509証明書の利用手順

LPIC303では、以下の各コマンドの使い方が問われる
- httpd.conf の記述
- mod_ssl
- openssl


OpenSSLの利用例:mod_ssl


証明書ファイルの指定
/etc/httpd/conf.d/ssl.conf に、サーバ証明書、ペアとなる秘密鍵、中間CA証明書のファイルを指定
過去には中間CA証明書は SSLCertificateChainFileで指定したが現在は中間CA証明書とSSL証明書を合わせて SSLCertificateFile に設定

SSL/TLSプロトコルの指定

```
# TLS v1.2のみ許可
SSLProtocol +TLSv1.2
# SSLv2/v3を禁止
SSLProtocol all –SSLv2 –SSLv3
```

HSTS HTTP Strict Transport Security
HTTPスキーマにアクセスがあった場合、HTTPSに差し替える仕組み

SNI Server Name Indication
一つのIPで複数のドメイン名を対応させるときの仕組み






### ある事業者でのSSL証明書更新の流れ

RHEL系の場合

```
sudo openssl genrsa -out /etc/pki/tls/private/localhost.key 2048

sudo cp -p /etc/pki/tls/certs/server.csr /etc/pki/tls/certs/server.csr.old

sudo openssl req -new -key /etc/pki/tls/private/localhost.key -out /etc/pki/tls/certs/server.csr

# チェックサム
openssl rsa --noout -modulus -in  /etc/pki/tls/private/localhost.key |openssl md5
openssl rsa --noout -modulus -in  /etc/pki/tls/certs/server.csr |openssl md5

# SSL証明書が届いたらこれもチェックサムを確認
sudo openssl x509 --noout -modulus -in /path/server.crt | openssl md5
```

Debian/Ubuntuの場合

```
sudo openssl genrsa -out /etc/ssl/private/localhost.key 2048

sudo openssl req -new -key /etc/ssl/private/localhost.key -out /etc/ssl/certs/server.csr

# チェックサム
sudo openssl rsa --noout -modulus -in  /etc/ssl/private/localhost.key |openssl md5
sudo openssl req --noout -modulus -in  /etc/ssl/certs/server.csr | openssl md5

# SSL証明書が届いたらこれもチェックサムを確認
sudo openssl x509 --noout -modulus -in /path/server.crt | openssl md5
```

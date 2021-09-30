---
title:331-OpenSSL
---

# 331-暗号化

## 331.1 OpenSSL

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

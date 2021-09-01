---
title:334-nikto
---
# 334-試験には出ないアプリケーション類
 
## nikto

 - Webサーバの脆弱性スキャナ

 -  KALIには初期状態で入っている

### Ubuntu18.04の場合

- インストール
  ```
  sudo apt install nikto libnet-ssleay-perl
  ```

### CentOSの場合

- インストール
  
  ```
  sudo dnf install nikto perl-Net-SSLeay
  sudo mkdir /usr/share/nikto/docs
  ```

### コマンド類

| 用途         | コマンド                                             |
| ---------- | ------------------------------------------------ |
| DB更新       | sudo nikto update                                |
| 対象サイトのスキャン | nikto -h 192.168.10.1 <br/> nikto -h www.any.lan |

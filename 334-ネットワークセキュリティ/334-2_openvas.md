---
title:334-openvas
---
# 334-ネットワーク侵入検知

## OpenVAS/GVM

- Nessus のFork 

- 303-300ではOpenVASが出題されるが、2021/7時点でEOL
- 雰囲気を味わうだけならUbuntu18.04(パターン更新できない)
- 後継のGVMでも一部コマンドは継承しているがopenvas-adduser,openvas-rmuserは存在しない。 gvmd gvm-manage-certs あたりは大体動作は同じ
- openvas用のfeed に feed.openvas.org が提供されていたが、2020.9で停止
- GVM用のfeedに feed.community.greenbone.net が提供されているが、GVM11用は停止済み

### Ubuntu 18.04の場合(参考)

- インストール
  ```
  sudo apt install openvas
  sudo openvas-setup 
  ```

  OpenVASの起動後、 https://localhost:9392 でOpenVASのWEBUIに接続できる

  実際にはGreenbone Security Assistant につながる

- 設定ファイル

  /etc/openvas/openvassd.conf 



### 主なユーティリティ(参考)

| コマンド名            | 用途                              |
| ---------------- | ------------------------------- |
| openvasmd        | OpenVAS Manager 各種操作            |
| openvassd        | OpenVAS Scanner スキャン処理          |
| openvas-nvt-sync | Network Vulnerability Test の更新  |
| openvas-mkcert   | 証明書の作成                                |
| openvas-adduser  | ユーザ追加<br>openvasmd create-user に同じ |
| openvas-rmuser   | ユーザ削除<br>openvasmd delete-user に同じ |
|                  |                                 |
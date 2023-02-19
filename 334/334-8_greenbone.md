---
title:334-Greenbone Security Assistant
---
# 334-ネットワーク侵入検知

## GVM

- Nessus のFork 

- 303-300ではOpenVASが出題されるが、2021/7時点でEO
- 後継のGVMでも一部コマンドは継承しているがopenvas-adduser,openvas-rmuserは存在しない。 gvmd gvm-manage-certs あたりは大体動作は同じ
- GVM用のfeedに feed.community.greenbone.net が提供されているが、GVM11用は停止済み

### Ubuntu 20.04の場合(参考)

- リポジトリを適切に追加すればインストール可能

  https://launchpad.net/~mrazavi/+archive/ubuntu/gvm

- インストール(一部)
  ```
  # 予めPostgreSQLが必要
  # GVMで使用するクレデンシャルの設定に注意...
  sudo apt install postgresql
  # リポジトリ追加
  sudo add-apt-repository ppa:mrazavi/gvm
  sudo apt install gvm
  ```

### 主なユーティリティ(参考)

| コマンド名            | 用途                              |
| ---------------- | ------------------------------- |
| gvmd        | Manager 各種操作            |
| openvas        | OpenVAS Scanner スキャン処理          |
| greenbone-nvt-sync | Network Vulnerability Test の更新  |
| gvmd --create-user  | ユーザ追加<br>openvasmd create-user に同じ |
| gvmd --delete-user   | ユーザ削除<br>openvasmd delete-user に同じ |
|                  |                                 |
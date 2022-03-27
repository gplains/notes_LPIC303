---
title: DebianでのPowerShell導入
---
# DebianでのPowerShell導入

## インストール

- パッケージリポジトリを登録する

- Powershell-lts-7.2.x.deb-amd64.deb を入手する
  ```
  sudo dpkg -i Powershell-lts-7.2.x.deb-amd64.deb
  ```

## アンインストール

- dpkgでインストールした場合/リポジトリの場合を問わず
  ```
  sudo apt-get remove powershell
  ```

## PowerCLIを使う場合

- 適当な方法でPowerCLIを入手してtarballにまとめる

- /opt/microsoft/powershell/7-lts/Modules/ 配下に配置する

  LTSの場合は7-lts/ になる。LTSじゃない場合は7/になる


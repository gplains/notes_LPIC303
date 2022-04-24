---
title:RHEL8の初期設定とチェックリスト
---

# RHEL8の初期設定とチェックリスト

あくまで2021年末までのメモ、それ以降は怖くて使えない

## インストール

- CDベースのインストール

  rhel-8.3-x86_64-boot.iso を使用したネットワークインストールの場合

  予めNICを有効化,IPアドレスをDHCP払出か設定する(これやらないと以降ができないみたい)

  RHSMに登録したID/パスワード (あるいは、サブスクリプションID/識別名)でサブスクリプションを有効化

  そのあとでリポジトリを設定(CentOS8/8Streamでは参照先フルパスの指定必要)
  
  仮想マシンの場合、 minimal + 標準 + ゲストエージェント の組み合わせが最低限必要(業務で使う場合)

### 実際

- 言語とロケーション 日本語

- システム > インストール先　

  ストレージ設定のカスタマイズ

  標準パーティション
  - /boot/efi 1GB
  - swap 8GB
  - / 残り全部

- システム > ネットワークとホスト名

  ホスト名 任意

  対象NICを有効化 DHCPのときIP払い出し

- Software > Connect to Red Hat

  Red Hatへの接続

  Authentication Method

  - アカウント認証:カスタマポータルのユーザ名、パスワード
  - アクティベーションキー: 組織ID および アクティベーションキー

- 参照リポジトリ 
 
  CentOS 8 Stream の場合

  http://ftp.riken.jp/Linux/centos/8-stream/BaseOS/x86_64/os

  RHEL8 の場合

  ユーザ名、パスワードを指定して Redhat CDN

- ソフトウェア > インストールソース

  最小限のインストール

  - ゲストエージェント
  - 標準

- ユーザの設定 > rootパスワード

  ユーザの作成 で管理者にチェックを入れる場合は省略可能

- ユーザの設定 > ユーザの作成

  このユーザを管理者にする

  usermod -G wheel とほぼ同じ

## チェックリスト

```
# 動作レベル
systemctl get-default

# サービス一覧
systemctl list-unit-files --type service --no-pager
# サービス一覧をタブ区切りに
systemctl list-unit-files --type service --no-pager | sed -e "s/  */\t/g"

# ディスクレイアウト
df -h

# 言語、ロケーション
localectl status
localectl set-locale LANG=ja_JP.UTF-8 #日本語でUTF-8に変更

# 時刻
timedatectl status
timedatectl set-timezone Asia/Tokyo # UNCから東京に変更
timedatectl set-local-rtc 0

# ネットワーク
ip a; nmcli con show ; nmcli conshow <NIC> | grep ipv4
lsof -i 
netstat -nr

# ファイアウォール
nft lisst ruleset ; lsmod | grep nf
firewall-cmd --list-all

# NTP
cat /etc/chrony.conf
systemctl status chronyd.service

#sysctl で設定する制約
cat /etc/sysctl.d/***.conf

#ssh
cat /etc/ssh/sshd_config 

#ホスト名
uname -a ; hostname
hostnamectl set-hostname #ホスト名の設定

# Hyper-V ゲスト
lsmod |grep hv

#パッケージ一覧
rpm -qa | sort

# ログ整形
ls -l /etc/rsyslog.d/+.conf
ls -l /etc/logrotate.d/*

# 監査
/etc/audit/auditd.cond # auditd.rules も
# sudoer
/etc/sudoers /etc/sudoers.d/
#cron
/etc/crontab /var/spool/cron /etc/cron.d/
#kdump
/etc/kdump.conf
#名前解決
/etc/hosts /etc/resolv.conf

#個別のサービス確認
systemctl is-enabled someservice

#ネットワークファイル
/etc/sysconfig/network
/etc/sysconfig/network-scripts/

#システム上限
/etc/security/limits.conf
/etc/security/limits.d/90-nproc.conf

# メモリ,CPU
free
/proc/cpuinfo
uname -n
/etc/redhat-release

#GRUB
/etc/grubd.d/40-custom
sudo grub2-mkconfig -o /boot/grub2/grub.cfg # 起動コンフィグに反映

# ストレジ
fdisk -C 
/etc/fstab

# SELINUX
sudo getenforce
sed -ie "s/\(SELINUX=\)enforcing/\1disabled/" /etc/selinux/config 

# サブスクリプション
# RHCPで登録したシステムは4時間ごとにポーリングされる
sudo subscription-manager status
sudo dnf group list 
```

## Tips

- 一括サービス確認
  ```
  # サービス確認したいサービスをservicelist として列挙しておく
  vi servicelist 
  # 以下内容
  nftables
  firewalld
  dhcp
  httpd
  chrony 

  # systemctl をgrep
  sudo systemctl list-unit-files | grep -f services 
  ```


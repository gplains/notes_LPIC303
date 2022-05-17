
---
title:CentOS7でZABBIX5をインストールする(2022/5)
---
# CentOS7でZABBIX5をインストールする(2022/5)

## 目的

- ひとまずzabbix5の使用感に慣れる

  RHEL7クローンの場合：Zabbix 5.0LTSまでで打ち止め(5.4もzabbix-serverとしては使えない)

  RHEL8クローン、Ubuntu18.04以降の場合: Zabbix 6.0LTSも利用可能

## 参考にしたURI

- Zabbix 3.0をCentOS 7にインストール

  https://qiita.com/atanaka7/items/294a639effdb804cfdaa

- CentOS 8 で Zabbix5.0にアップグレード(nginx + PHP-FPM + MySQL)

  https://kusoneko.blogspot.com/2020/10/centos-8-upgrade-to-zabbix5.0.html

- [zabbix 4.0] アクティブチェックが失敗するときの確認点
  
  https://mseeeen.msen.jp/check-point-when-zabbix-active-check-fails/

- Partitioning a Zabbix MySQL(8) database with Perl or Stored Procedures
  
  https://blog.zabbix.com/partitioning-a-zabbix-mysql-database-with-perl-or-stored-procedures/13531/


## KVM親機でやること

- 適当な方法でCentOS7イメージを作成する

  省略...と云いたいところだが、適当なところでスナップショットを切る(VMにc7zbxとした場合)

  ```
  # スナップショット採取
  sudo virsh snapshot-create-as --domain c7zbx --name base
  # スナップショットに戻す
  sudo virsh snapshot-revert --domain  c7zbx --current
  ```

## 実際にzabbix5サーバを立てる

- CentOS7イメージにMySQL8.0とPHP7.3を適用する
  ```
  # CentOS用SCL投入
  sudo yum install centos-release-scl
  sudo yum list  rh-php7\*
  
  # CentOS7用MySQL8.0投入
  sudo rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
  sudo yum repolist all | grep mysql
  sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
  sudo yum install mysql-community-server
  # mariadb-libs.x86_64 1:5.5.68-1.el7 が削除される
  
  # 意味もなくMySQLを起動する
  sudo systemctl start mysqld.service
  ```

- MySQLのパスワード回りをごにょごにょする
  ```
  # 初期パスワードを確認
  sudo grep password /var/log/mysqld.log
  
  # パスワードポリシを修正する
  sudo vi /etc/my.cnf
  # 以下の文字列を追記
  # validate_password.policy=LOW
  # validate_password.length=4
  # validate_password.check_user_name=OFF
  
  # 一回だけ再起動しておく
  sudo systemctl start mysqld.service
  
  # 初期パスワードでmysqlにログイン
  mysql -u root -p
  # 以下のコマンドを入力
  # ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootpassword';
  # show variables like 'validate_password%';
  # quit ;
  
  # ついでにZABBIX用のスキーマ定義も書いておく
  mysql -u root -p
  # 以下のコマンドを入力
  # create database zabbix character set utf8 collate utf8_bin;
  # create user 'zabbix'@'localhost' identified WITH mysql_native_password  by 'zabbixpw';
  # grant all on zabbix.* to 'zabbix'@'localhost' with grant option;
  # FLUSH PRIVILEGES;
  # quit ;
  ```

- ZABBIXのリポジトリを設定する
  ```
  # ZABBIXのリポジトリを登録
  sudo rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
  # /etc/yum.repos.d/zabbix.repo を修正、 frontend区を有効にする
  sudo vi /etc/yum.repos.d/zabbix.repo 
  ```

- 適宜インストール
  ```
  # リポジトリDBを更新
  sudo yum update
  # ますPHP73をインストール
  sudo yum install zabbix-web-mysql-scl-php73  zabbix-apache-conf-scl
  
  # fpmの時刻設定を変更
  cd /etc/opt/rh/rh-php73/php-fpm.d/
  sudo vi zabbix.conf
  # php_value[date.timezone] = Asia/Tokyo
  
  # ZABBIXをインストール
  yum -y install zabbix-server-mysql zabbix-agent zabbix-web-japanese
  # 設定ファイルを修正
  cd /etc/zabbix/
  sudo vi zabbix_server.conf
  # DBPassword 区を修正

  # 次にhttpdの設定
  cd /etc/httpd/conf.d/
  sudo cp -p zabbix.conf zabbix.conf.org
  sudo vi zabbix.conf
  # SetHandler のphp72 の個所をコメントアウト
  # SetHandler のphp73 の個所のコメントアウトを外す
  ```
  
 - 各サービスを起動
   ```
   sudo systemctl start zabbix-server
   sudo systemctl start zabbix-agent
   sudo systemctl start httpd
   sudo systemctl enable zabbix-server
   sudo systemctl enable zabbix-agent
   sudo systemctl enable httpd
   sudo systemctl status zabbix-server
   sudo systemctl status zabbix-agent
   ```
- SELinuxに筋を通す
  ```
  sudo getsebool -a | grep zabbix # 多分した三つだけ表示される
  sudo setsebool -P httpd_can_connect_zabbix on
  sudo setsebool -P zabbix_can_network on
  sudo setsebool -P zabbix_run_sudo on
  ```
 
- RHEL系特有のSELINUXのアレ
  ```
  sudo grep zabbix_server /var/log/audit/audit.log | audit2allow -M zabbix-limit
  sudo semodule -i zabbix-limit.pp
  sudo grep AVC /var/log/audit/audit.log | audit2allow -M systemd-allow
  sudo semodule -i systemd-allow.pp
  ```

- RHEL7特有のFirewalldのアレ(やっつけ)
  ```
  # 面倒であればデフォルトルールに以下ぶちまけてもいい、その場合add-source区は見直し
  sudo firewall-cmd --permanent --new-zone zabbix
  sudo firewall-cmd --permanent --zone=zabbix --add-service=http
  sudo firewall-cmd --permanent --zone=zabbix --add-port=10050
  sudo firewall-cmd --permanent --zone=zabbix --add-port=10050/tcp
  sudo firewall-cmd --permanent --zone=zabbix --add-port=10051/tcp
  sudo firewall-cmd --permanent --zone=zabbix --add-source=192.168.0.0/16
  sudo firewall-cmd --reload
  sudo firewall-cmd --list-all --zone=zabbix
  
  ```

## プラグイン回り

- java-gateway 

  ```
  # zabbixが設定されていればyum一発
  sudo yum install zabbix-java-gateway
  ```

## パーティショニング

基本的には、前述の「Partitioning a Zabbix MySQL(8) database with Perl or Stored Procedures」を読んで進めればOK

ただ、雰囲気からするとアレです

60日残す設定だと「一回目のアレ」のhistory_uintも60世代作る必要がありますし

history_uintが100GBくらいあると、もしかすると1時間くらいパーティショニング終わらないかも

あとディスクが元のDBの2倍要るところが注意

- 一回目のアレ
  ```
  mysql -u root -p zabbix

  # 以下一気にコピペ(history_uint)
  ALTER TABLE history_uint PARTITION BY RANGE ( clock)
  (PARTITION p2022_05_16 VALUES LESS THAN (UNIX_TIMESTAMP("2022-05-17 00:00:00")) ENGINE = InnoDB,
  PARTITION p2022_05_17 VALUES LESS THAN (UNIX_TIMESTAMP("2022-05-18 00:00:00")) ENGINE = InnoDB
  );

  # 以下一気にコピペ(trends)
  ALTER TABLE trends_uint PARTITION BY RANGE ( clock)
  (PARTITION p2022_04 VALUES LESS THAN (UNIX_TIMESTAMP("2022-05-01 00:00:00")) ENGINE = InnoDB,
  PARTITION p2022_05 VALUES LESS THAN (UNIX_TIMESTAMP("2022-06-01 00:00:00")) ENGINE = InnoDB
  );

  quit;
  ```

- 日次で分割してくれるシェル
  
  https://github.com/OpensourceICTSolutions/zabbix-mysql-partitioning-perl

  先述の「alter table」を仕込んだテーブルだけパーティション追加削除してくれる

  逆に言うと、alter tableでパーティショニングしてないテーブルは新規で処置したりはしないので注意(最初の一回はmysqlコマンドを叩く必要がある)

  先述の記事にある通り、適当な場所にperlスクリプトを配置してcronで毎日実行させること
  
  ```
  # 予め依存関係を解決しておく(MySQL/MariaDBどっちでもOK)
  sudo yum install perl-DateTime perl-Sys-Syslog perl-DBI perl-DBD-mysql
  vi mysql_zbx_part.pl
  sudo mv mysql_zbx_part.pl /usr/share/zabbix
  cd /usr/share/zabbix
  chmod +x mysql_zbx_part.pl

  # cron で /usr/share/zabbix/mysql_zbx_part.pl を実行する
  sudo crontab -e 
  ```

  簡単に修正箇所をまとめる(MySQL8.0の場合。MariaDBの場合は94-97行目をコメントアウト解除して、98-103行目はそのまま)
  ```
  # 12行目
  my $db_password = 'DBPassword に書いたzabbixのパスワード';

  # 32行目
  my $curr_tz = 'Asia/Tokyo';

  # 86行目-93行目
  MySQL8の場合は全部コメントアウト

  # 98行目-103行目
  MySQL8の場合はコメントアウト解除

  # 186行目
  ZABBIX5.0の場合はコメントアウト解除
  ```

  Perlスクリプトに実行フラグ立ててcronで回せば、多分勝手に分割してくれるはず

## バックアップどうしよ

- 安直に: mysqldump -u root -p zabbix

- 安全方向に倒す場合: ZABBIXサポートに契約して、バックアップスクリプトをわけてもらう

- 海外の有志のスクリプトを使う

  https://github.com/remontti/zabbix-backup
---
title:333-SELinux
---

# 333-アクセスコントロール

## SELinux

- RHEL7/8 および、これらのクローンでの実施を推奨
  
  (Mastering Linux Security and Hardening)
- Ubuntu20.04 では少し具合が悪い(仮想マシン上でインストール再起動すると高確率でフリーズする)


## Ubuntu 20.04 の場合

※Ubuntuの場合、唐突にenforcing にすると詰むのでpermissive で色々試しましょう

- インストール

  ```
  # apparmor を完全削除 、して再起動
  sudo systemctl stop apparmor
  sudo apt remove apparmor -y
  sudo reboot
  # auditdをインストール
  sudo apt install auditd  # audit2why他でお世話になるので
  # selinux一式をインストール
  sudo apt install selinux-basics selinux-policy-default selinux-utils setools selinux-policy-src
  sudo selinux-activate
  sudo reboot

  # 状態を確認
  sestatus  # permissive が返る...はず
  sudo ls -l /var/log/audit # audit.log が出力されている

  # ポリシ不一致で大量にエラーが出るのでとりあえず黙らせる
  sudo audit2allow -M somepolicy < /var/log/audit/audit.log
  sudo semodule -i somepolicy.pp  # ポリシ反映
  sudo reboot
  ```



## CentOS 8.x の場合

- インストール

   標準でインストール済み



## 各種コマンド

- 状態確認

  ```
  # 有効無効の確認
  getenforce
  # 詳細な状態の確認
  sestatus
  # selinux の統計を出力
  seinfo
  ```


- モード変更
  ```
  # 一時的な変更
  sudo setenforce [Permissive|enforcing]
  # 恒久的な変更
  sudo sed -ie "s/^\(SELINUX=\)enforcing/\1permissive/" /etc/selinux/config
  sudo reboot
  ```

- SELinuxに関する各種操作

  ```
  # SELinuxに関する管理全体を司る
  semanage
  ```

- コンテキスト
  
  ```
  # セキュリティコンテキストの変更
  chcon
  # ファイルのセキュリティコンテキストを復元
  restorecon
  # 指定したセキュリティコンテキストでコマンドを実行
  runcon

  # SELinuxの設定ファイルに従ってセキュリティコンテキストを変更
  fixfiles
  # 指定ファイルに従ってセキュリティコンテキストを変更
  setfiles
  
  # ブール値を表示 
  getsebool -a # すべて表示
  getsebool [指定パラメタ] # 指定のパラメタに対するブール値を表示
  # ブール値を変更
  setsebool [指定パラメタ] {on|off} 
  togglesebool # ON|OFFの切り替え
  ```

- ポリシ解析等

  ```
  # 要setools-gui
  apol 
  # RHEL8にはない？SELINUXのログを監査
  seaudit
  # RHEL8にはない？SELINUXのログを監査しレポート出力
  seaudit-report
  # 監査ログから原因を出力
  audit2why
  # 監査ログから対策を表示
  audit2allow
  ```


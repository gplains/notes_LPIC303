---
title:rsyslog
---

# rsyslog

auditdと一緒に理解したい

Ubuntu/RHELとも、minimalインストールの時点で含まれる

- 設定ファイル

  /etc/rsyslog.conf

  /etc/rsyslog.d/somefile.conf

- ライフサイクル

  ```
  # 設定の編集
  sudo vi /etc/rsyslog.conf
  # コンフィグチェック
  sudo rsyslogd -N 1
  # 反映
  sudo systemctl restart rsyslog 
  ```

## 利用時の注意点





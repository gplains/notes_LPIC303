---
title:logrotate
---

# logrotateの簡単な理解

あくまで設計するときのアンチョコ


### Ubuntuの場合

- インストール
  
  標準でインストール済み


### CentOSの場合

- インストール
  
  標準でインストール済み

### 実際

- 設定ファイル
  
  /etc/logrotate.conf
  
  /etc/logrotate.d/somepolicy

  ※ /etc/logrotate.d/ にバックアップのつもりでファイルを配置すると実行されるので

  ※ /etc/logrotate.bak/ のように別フォルダを切ってそちらに退避しよう

- 流れ

  ```
  # 設定変更
  sudo vi /etc/logrotate.d/example
  # コンフィグチェック
  sudo logrotate -dv /etc/logrotate.d/example
  # 設定反映
  (翌朝のジョブを確認) 
  ```


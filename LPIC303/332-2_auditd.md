---
title:332-auditd
---

# 332-ホストの侵入検知

## auditd

- RHEL7/8 デフォルトで有効
- Ubuntu20.04 別途apt等でインストール必要

## RHEL7/8 の場合

- インストール
  
  標準でインストール済み


## Ubuntuの場合

- インストール

   ```
   sudo apt install auditd
   sudo systemctl restart auditd
   ```

## 一般的な用途

- 監査ルールの一覧

  ```
  sudo auditctl -l
  ```

- ファイル変更の検知

  ```
  # ルールを仕込む(揮発性)
  sudo auditctl -w /etc/passwd -p wa -k passwd_changes
  # w : パス
  # p : WARX = Write Attr Read eXcute 
  # k : ルール名
  
  # ルールを仕込む(揮発ルールを再起動後も有効に)
  sudo auditctl -l  # 揮発ルールの一覧を表示
  sudo cat /etc/audit/rules.d/custom.rules
  sudo sh -c "auditctl -l >> /etc/audit/rules.d/custom.rules"  # sh -c でリダイレクトするのが重要
  sudo cat /etc/audit/rules.d/custom.rules  
  sudo systemctl restart auditd.service
  sudo auditctl -l  # 揮発ルールに反映されたことを確認
  ```

- 検索とレポート

  ```
  # レポート
  sudo aureport 
  
  # 検索
  ausearch  
  ```


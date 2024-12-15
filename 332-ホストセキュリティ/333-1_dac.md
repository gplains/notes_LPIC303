---
title:332-OpenSCAP
---

# 332-ホストセキュリティ

## OpenSCAP

Security Content Automation Protocol 



## インストール

- Ubuntuの場合

  ```
  sudo apt install openscap-scanner scap-security-guide 
  ```

- RHEL系の場合

  ```
  sudo dnf install python-openscap bzip2
  ```

- スキーマ
  
  /usr/share/openscap/schemas

- CPE

  /usr/share/openscap/cpe 
 

## 各種コマンド

- oscap (オプション) (モジュール) (操作)
  
  oscap -h で確認しておこう
  
  - oval Open Vulnerability and Assessment Language
  - xccdf eXtensible Configuration Checklist Description Format
  - cvss Common Vulnerability Scoring System
  - CPE Common Platform Enumeration

## 実際の流れ

```
# 1.ovalをダウンロード
wget rhel-8.oval.xml.bz
# 2.yumで使用するツールを投入
yum install openscap-utils bzip2 
# 3.リモートの対象にopenscap-scanner を投入
# 4.実際に流す
oscap-ssh user@host --report report-oval.xml


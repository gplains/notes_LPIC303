---
title:334-wireshark/tshark
---
# 334-ネットワークハーデニング

## Wireshark/tshark

- wiresharkパッケージに同梱
- GUIがwireshark、CUIがtshark
- フィルタ定義は同じ

### Ubuntu 18.04の場合

- インストール
  ```
  sudo apt install wireshark  # tsharkのみの導入は不可
  ```

### RHEL/CentOSの場合

- インストール
  ```
  sudo dnf install wireshark 
  ```

### tshark の引数

| パラメタ | 項目                  |
| ---- | ------------------- |
| D    | 有効なネットワークインタフェースの一覧 |
| i    | 監視するインタフェース         |
| f    | フィルタ                |
| r    | 読み込み元               |
| w    | 保存先                 |
| V    | 詳細表示                |
| z    | 項目に対する詳細情報                    |

### tshark/wiresharkのフィルタ式
- host <ホスト>
- net <ネットワーク>
- (tcp|udp) prot <port#>
- src (送信元)
- dst (送信先)
- portrange A-B

| 記述      | 数式     |
| ------- | ------ |
| A eq B  | A == B |
| A gt B  | A > B  |
| A ge B  | A >= B |
| A lt B  | A < B  |
| A le B  | A <= B |
| A and B | A && B |
| A or B  | A \|\| B       |


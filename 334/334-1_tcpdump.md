---
title:tcpdump
---
# 334-ネットワークハーデニング

## tcpdump

- フィルタ定義はwiresharkに同じ


### tcpdump の引数

| パラメタ    | 項目                  |
| ------- | ------------------- |
| i <int> | 指定したインタフェースを監視      |
| n       | ホスト名の代わりにIPアドレスを表示  |
| nn      | プロトコル、ポート番号を変換なしに表示 |
| r       | 読み込み元               |
| w       | 保存先                 |
| s       | パケットから取り出すバイト数      |
| x       | 内容を16進数で表示          |
| X       | 内容を16進数+ASCIIで表示    |
| p       | プロミスキャスモードに"しない"    |
| m       | MIBモジュールを読み込む                    |

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


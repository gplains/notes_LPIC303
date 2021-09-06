---
title:334-StrongSwan
---

# 334-VPN


## StrongSwan

- Ubuntuの場合、比較的容易にインストール/設定が可能

## 参考にしたもの

- StrongSwanの公式記事

  https://www.strongswan.org/testing/testresults/ikev2/net2net-pubkey/index.html

### Ubuntu 20.04 の場合

- 今回はサイト間VPNを考える。検証環境の場合、サイト間のパブリックIP同士、サイト１のプライベート側、サイト２のプライベート側で都合3つVLANを切る
  
  サイト１
  | セグメント   | IPアドレス            |
  | ------- | ----------------- |
  | パブリック側  | 198.51.100.100/24 |
  | プライベート側 | 192.168.10.1/24   |
  
  サイト２
  | セグメント   | IPアドレス           |
  | ------- | ---------------- |
  | パブリック側  | 198.51.100.101/24 |
  | プライベート側 | 192.168.20.1/24  |

- インストールに際して、別途いくつかのノードを作成する

  ルータノード
  | セグメント | IPアドレス          |
  | ----- | --------------- |
  | サイト１側 | 198.51.100.1/24 |
  | 管理/外抜け| 任意(192.168.81.xx/24) |
 
  サイト１子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.10.100/24       |

  サイト2子ノード
  | セグメント   | IPアドレス |
  | ------- | ------ |
  | プライベート側 | 192.168.20.100/24       |

- インストール(サイト1/サイト2とも)
  
  ```
  # 予めip_forwardを許可
  sudo vi /etc/sysctl.conf
  > net.ipv4.ip_forward=1
  sudo apt install strongswan  strongswan-swanctl strongswan-pki
  ```

- 以下2例とも共通.../etc/sstongswan.d/charon.conf でswanctlを呼ぶ
  ```
  sudo vi /etc/strongswan.d/charon.conf
  # ...中略
  start-scripts {
      swanctl = /usr/sbin/swanctl -q  # 追記
  }
  ```
## 事前共有鍵(PSK)の場合

設定ファイルの作成(サイト1の場合、サイト2は一部修正要)

IKE/ESP のプロポーザルは適宜変更しましょう

```
# /etc/swanctl/conf.d/site2site-psk.conf

connections {
    site-2-site {
        version = 2
        proposals = aes256-sha256-modp2048,default
        local_addrs = 198.51.100.100    # サイト1のパブリックIP
        remote_addrs = 198.51.100.101   # サイト2のパブリックIP
        local {
            auth = psk
            id=198.51.100.100           # サイト1のパブリックIP
        }
        remote {
            auth = psk
            id=198.51.100.101           # サイト2のパブリックIP
        }
        children {
            site-2-site {
                local_ts = 192.168.10.0/24  # サイト1のローカルIP
                remote_ts = 192.168.20.0/24 # サイト2のローカルIP
                updown = /usr/local/libexec/ipsec/_updown iptables
                mode=tunnel
                esp_proposals = aes256-sha256-modp2048,default
            }
        }
    }
}
secrets {
    ike-1 {
        secret = "PRE_SHARED_KEY"       # 事前共有鍵は1/2で同一
    }
 }
```

## 公開鍵(pubkey)の場合


- 秘密鍵/公開鍵の作成(サイト1/サイト2とも)
  
  ```
  # router-l
  ipsec  pki --gen -s 4096 --outform pem  > router-l.key
  ipsec pki --pub  --outform pem < router-l.key  >router-l.pub
  scp router-l.pub userid@198.51.100.101:~/
  # router-r
  ipsec  pki --gen -s 4096 --outform pem  > router-r.key
  ipsec pki --pub  --outform pem < router-r.key  >router-r.pub
  scp router-r.pub userid@198.51.100.100:~/
  ```

- 秘密鍵/公開鍵の配置(サイト1/サイト2とも)

  ```
  #router-l
  sudo cp router-l.key /etc/swanctl/private/
  sudo cp router-[lr].pub /etc/swanctl/pubkey/
  #router-r
  sudo cp router-r.key /etc/swanctl/private/
  sudo cp router-[lr].pub /etc/swanctl/pubkey/
  ```

StrongSwan設定ファイルの作成(サイト1の場合、サイト2は一部修正要)

IKE/ESP のプロポーザルは適宜変更しましょう

```
# /etc/swanctl/conf.d/site2site-pubkey.conf
connections {
    site-2-site {
        version = 2
        proposals = aes256-sha256-modp2048,default
        local_addrs = 198.51.100.100     # サイト1のパブリックIP
        remote_addrs = 198.51.100.101    # サイト2のパブリックIP
        local {
                auth=pubkey
                id=198.51.100.100        # サイト1のパブリックIP
                pubkeys=router-l.pub     # サイト1の公開鍵
        }
        remote {
                auth = pubkey
                id=198.51.100.101        # サイト2のパブリックIP
                pubkeys=router-r.pub     # サイト2の公開鍵
        }
        children {
            site-2-site {
                local_ts = 192.168.10.0/24  # サイト1のローカルIP
                remote_ts = 192.168.20.0/24 # サイト2のローカルIP
                updown = /usr/local/libexec/ipsec/_updown iptables
                mode=tunnel
                esp_proposals = aes256-sha256-modp2048,default
            }
        }
    }
}
```

## サービス有効化(サイト1/サイト2とも)

  ```
  systemctl restart strongswan-starter.service
  swanctl -L
  swanctl -q
  swanctl -L
  swanctl --initiate --child site-2-site
  ```

### CentOS8 の場合

- 非推奨(RHEL8ではlibreswanのみサポート)のため省略

## 引っかかりどころ

- ネットワーク構成

  Linuxルータを両ノードで挟む構成の場合、NAT越えに失敗して "NO_PROPOSAL_CHOSEN" を返すことがある

  できれば、両ノードはL2SWを挟む程度(なるべく直結)にすること。

  ```
  Sep  6 15:33:02 router-l charon: 05[NET] received packet: from 198.51.100.1[500] to 198.51.100.100[500] (1172 bytes)
  Sep  6 15:33:02 router-l charon: 05[ENC] parsed IKE_SA_INIT request 0 [ SA KE No N(NATD_S_IP) N(NATD_D_IP) N(FRAG_SUP) N(HASH_ALG) N(REDIR_SUP) ]
  Sep  6 15:33:02 router-l charon: 05[IKE] no IKE config found for 198.51.100.100...198.51.100.1, sending NO_PROPOSAL_CHOSEN
  Sep  6 15:33:02 router-l charon: 05[ENC] generating IKE_SA_INIT response 0 [ N(NO_PROP) ]
  Sep  6 15:33:02 router-l charon: 05[NET] sending packet: from 198.51.100.100[500] to 198.51.100.1[500] (36 bytes)
  ```

- 設定ファイル

  所謂stroke(ipsec.conf)向けの手順が大半で、今Ubuntu 20.04でインストール可能なStrongSwan  5.8の場合は vici(swanctl.conf)の手順が妥当とのこと...
  

## 今回の検証で困ったこと

- 何かいい作図ツールないかなぁ...

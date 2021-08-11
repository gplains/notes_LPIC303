# CentOS8でLibreSwanのサンプル

## 参考にしたもの
- 技術メモの壁

  https://fsck.jp/?p=972
- RHEL8 第4章 IPSEC を使用した VPN の設定

  https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/8/html/securing_networks/configuring-a-vpn-with-ipsec_securing-networks#creating-a-host-to-host-vpn_configuring-a-vpn-with-ipsec

## 引っかかりどころ


- 経路設定(踏み台)

  NW構成は「技術メモの壁」を参考にしたのですが、思ったよりlibreswanの設定ファイル修正がきつかったです。
  今回はルータ(てっぺん)にSSH接続できるようにして、これを踏み台にして各VPNノードにSSH接続できるようにしました。ていうかこんなのVMware Workstation のウィンドウから入力できねえ。

- ipsecコマンドの仕様変更

  一部のコマンド引数が変わってしまった

  ```bash
  # 従前の例
  ipsec newhostkey --output /etc/ipsec.d/hostkey.secrets
  # 実際の例
  ipsec newhostkey
  > Generated RSA key pair with CKAID (...) was stored in the NSS database
  > The public key can be displayed using: ipsec showhostkey --left --ckaid (...)```

- leftとかrightとか

  自分(local)=left 、対抗(remote)=right 。

  さっきの ipsec newhostkeyも、対抗で –left オプションで出力しといてrightrsasigkeyにコピペしとけばよし。逆もまた然り。

## 今回の検証でわかったこと

- (上に書いた通り)一部引っかかる個所もあるが、基本的には手順通りでLibreswanの環境が構築できる
- ベースイメージさえ作っておけば”リンククローン”でばんばん増やせる(たぶんProだけの機能)
- StrongSwanはそもそもepelだし本家RHELではサポート外
- wireguard-toolsはそもそもepelだし本家RHELではサポート外
- openvpnはそもそもepelだし本家RHELではサポート外
- KAMEなんざ泳いでねーよ(知ってた)

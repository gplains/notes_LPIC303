
---
title:ZABBIXでExtreme VDXのREST機能を利用する
---
# ZABBIXでExtreme VDXのREST機能を利用する

## 目的

- Extreme VDXで諸事情でSNMPが無効化されている場合にRESTでなんとか採取する

## 順序

- REST問い合わせするタネを作る
- ZABBIX用にスクリプトをこさえる
- zabbix_agent用設定ファイルをこさえる
- ZABBIX WEBUI で実際に登録する

## REST問い合わせするタネを作る

以下Linuxの場合。けっこう疑似コード

```
 VDXIP=10.1.2.3
 INTERFACE=$1  # 1/0/1 とか
 REQBODY = "<get-interface-detail><interface-type>tengigabitethernet</interface-type> \
           <interface-name>$INTERFACE</interface-name></get-interface-detail>"
 RETVAL=`curl --user admin:password http://$VDXIP/rest/operational-state/get-interface-detail -d $REQBODY
 echo $RETVAL
 ```

まず、これで ``` sh vdxquery.sh 1/0/1 ``` なんて流してみて、1/0/1に対する応答が返ればOK

あとは、レスポンスに対してxpathを書く...これがしんどい

```
# ifHCInOctets
sh vdxquery.sh 1/0/1   | xmllint --xpath '//*[local-name()="ifHCInOctets"]/text()' -
# ifHCOutOctets
sh vdxquery.sh 1/0/1   | xmllint --xpath '//*[local-name()="ifHCOutOctets"]/text()' -
```

## ZABBIX用にスクリプトをこさえる

一個のシェルで書くとこんな感じ

格納先は、 /usr/lib/zabbix/externalscripts/

``` 
#!/bin/bash
# /usr/lib/zabbix/externalscripts/ifHCInOctets.sh
 VDXIP=10.1.2.3
 INTERFACE=$1  # 1/0/1 とか
 REQBODY = "<get-interface-detail><interface-type>tengigabitethernet</interface-type> \
           <interface-name>$INTERFACE</interface-name></get-interface-detail>"
 RETVAL=`curl --user admin:password http://$VDXIP/rest/operational-state/get-interface-detail -d $REQBODY
 echo $RETVAL  | xmllint --xpath '//*[local-name()="ifHCInOctets"]/text()' -
```

## zabbix_agent用設定ファイルをこさえる

/etc/zabbix/zabbix_agentd.d/VDX.conf
```
UserParameter=VDX.ifHCInOctets[*],/usr/lib/zabbix/externalscripts/ifHCInOctets.sh $1
```

zabbix-agent に"VDX.ifHCInOctets[1/0/1]" って問い合わせがあれば、/ifHCInOctets.sh 1/0/1 の結果が返るはず...

## ZABBIX WEBUI で実際に登録する

- 設定 > ホスト > アイテム の順に遷移

- 「アイテムの作成」

- 以下のように設定

  名前: UserParameterに指定した名前 "VDX.ifHCInOctets[1/0/1]"

  タイプ:Zabbix エージェント

  キー: UserParameterに指定した名前 "VDX.ifHCInOctets[1/0/1]"

  データ型:数値(整数) (UINTの場合)

  アプリケーション: 設定したほうがよい(VDXとか)

- 「保存前処理」で以下のように設定

  変化 > 1秒あたりの差分

  乗数: 8

## 別解:zabbix_sender を使う

zabbix_sender の場合は順序が若干異なる

今回のケース(1回curlすると In/Out両方のクエリが書ける)ではcurlの実行回数を減らせるので

もしかするとこっちのほうがいいかもしれない...

- zabbix_sender のインストール
  
  ```
  # 一応 zabbix-get も入れておく
  yum install zabbix-sender zabbix-get 
  ```

- ダミーホストの追加

```
設定>ホスト>「ホストの作成」
- 名前:任意(zabbix_senderで困らない程度に)
- 表示名:任意
- グループ:任意
- インタフェース:Zabbix エージェント(127.0.0.1)
- テンプレート:指定しない(ここ重要)
```

- アイテムの追加

  ```
  「アイテムの作成」
  - 名前:VDX.ifHCInOctets[1/0/1]
  - タイプ: Zabbix トラッパー
  - キー:VDX.ifHCInOctets[1/0/1]
  - データ型:数値(整数)
  - アプリケーション:設定したほうがよい
  - 保存前処理:以下略
  ```
- zabbix agent 側での設定

  「アイテムの作成」から1～2分寝かす必要があるようだ...

  ```
  # 疑似コード
  zabbix_sender -z localhost  -s "dummyVDX" -k "VDX.ifHCInOctets[1/0/1]" -o 1024768
  ```
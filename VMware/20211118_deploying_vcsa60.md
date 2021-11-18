---
title:VMware vCSA6.0のデプロイ
---
# VMware vCSA6.0のデプロイ

できないことはないが今更やることでもない、一応備忘として

# 参考にしたURI

- vCSA6.0のデプロイ手順

  https://www.virten.net/2015/04/how-to-install-vcenter-server-appliance-vcsa6-in-vmware-workstation/
- Nested ESXi
  
  http://vmwa.re/nestedesxi

## 準備するもの

- VMware Workstation の環境aa
- VMware-VCSA-all-6.0.0-14518058.iso

## 手順

- vcsaのインストールOVAを抽出する
  
  ISOイメージをマウント
  
  vcsaフォルダに遷移
  
  vmware-vcsa を vmware-vcsa.ova にリネーム

- VMware Workstation でOVAをインポート

  表示される項目は vCSA 6.7のときと概ね同じ

  ネットワーク設定、SSOのadministratorパスワード、あとシステムのrootパスワードの3個所は決めておこう

- 実際の操作
  
  CSharp Client(昔のWindows版Client)で、vSphere5.5相当の操作までは行える

  DataCenter作って、EVCクラスタやHAクラスタ作って、あとクラスタにホストを東麓するくらいならなんとか

## 参考:ESXiの安直なデプロイ

- 公式のnested esxiイメージを使う

  もちろんだがvCenter6.0の場合 ESXi 5.5/6.0 しか配下に置けない

  デフォルトではroot/VMware1! がID/PWになっているようだ

  一旦ネットワーク設定だけしてデプロイして、あとで8GBのストレージ領域を適当に拡張して使うと便利

  なほ、デフォルト設定だとログの出力先が揮発メモリになっているので「使い続ける場合は」少し面倒
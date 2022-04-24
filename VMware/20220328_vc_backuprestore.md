---
title:vCenter 6.5以降のバックアップとリストア
---
# vCenter 6.5以降のバックアップとリストア

vCenter 6.5以降は、バックアップリストアが従来の「イメージバックアップ」から「ファイルバックアップしてデプロイ後のイメージに対してリストア」に変わっている

一応vCenter6.7までは「イメージバックアップ/イメージリストア」でもよいが...はてさて


## バックアップ

- https://vcenter:5480 にアクセス

- [サマリ] > [管理] > [バックアップ] の順に遷移

- [アクティビティ] > [今すぐバックアップ] の順に遷移

  - バックアップの場所 schema://host:port/path
    
    6.7: FTP FTPS HTTP HTTPS NFS SMB SCP #SCVはSCP待ち受け不可

    7.0: FTP FTPS HTTP HTTPS NFS SMB SFTP 

  - バックアップサーバの資格情報

    - ユーザー名

    - パスワード

  - 暗号化バックアップ(オプション)

    暗号化パスワード(パスワードの確認)

※バックアップスケジュール
- スケジュール 日単位、　ｘ時ｘ分
- バックアップの数


## リストア

- Stage1 をインストール時と同様に行う

- stage2 の実行前に「バックアップ」で採取したファイルを適当な方法でローカルに複製する
  
  足が生えている先なっら、複製しなくてもいける

- [EULAに同意] > バックアップ情報を入力する

  - 場所またはIPアドレス/ホスト名

  - ユーザー名 : バックアップサーバの資格情報

  - パスワード : バックアップサーバの資格情報


---
title:KB76555対策を考えながらのESXiアップデート
---
# KB76555対策を考えながらのESXiアップデート

考えられるパターン

- 6.0u2未満 > 6.0u3 > ESXi600-201807001 > 6.7u3 > 7.0系
- 6.0u2未満 > 6.5u3GA > 7.0系


## 事前準備:バイナリのダウンロード

- オフラインバンドルのダウンロード

  https://customerconnect.vmware.com/jp/downloads/
- パッチのダウンロード

  https://customerconnect.vmware.com/patch

## 環境設定

```
# バージョン確認
esxcli system version get
#   Product: VMware ESXi
#   Version: 6.0.0
#   Build: Releasebuild-5050593
#   Update: 3
#   Patch: 57

# キャパ確認
df -h

# マウント状態確認
esxcfg-nas -l

# repo.nas.home の /datastore1 をマウント
esxcfg-nas  -a  -o repo.nas.home -s /datastore1 datastore

# 結果確認だけしておく
df -h
esxcfg-nas -l
```

## 設定のバックアップとリストア

予めvim-cmd でバックアップしておくとちょっとだけ幸せになれる

```
# バックアップ
esxcfg-info -u
vim-cmd hostsvc/firmware/sync_config
vim-cmd hostsvc/firmware/backup_config
# Bundle can be downloaded at : http://hostname/downloads/UUID/configBundle-host.tgz

# リストア
vim-cmd hostsvc/maintenance_mode_enter
vim-cmd hostsvc/firmware/restore_config /backup-location/configBundle.tgz
```


## VMKB76555 を踏み抜くテスト 

```
# バージョン確認
esxcli system version get
#   Product: VMware ESXi
#   Version: 6.0.0
#   Build: Releasebuild-5050593
#   Update: 3
#   Patch: 57

# OEMパッケージの有無確認。Nested ESXiではもちろん意味がない
esxcli software vib list |grep OEM

# vib install なほ KB76555を全力で踏み抜く
esxcli software vib install -d /vmfs/volumes/datastore/VMware-ESXi-7.0U3d-19482537-depot.zip
# [InstallationError]
# ('VMware_bootbank_trx_7.0.3-0.35.19482537', 'Could not find a trusted signer.')
#       vibs = VMware_bootbank_trx_7.0.3-0.35.19482537
# Please refer to the log file for more details.
```

## 6.0u3から無理矢理7.0にアップグレードするテスト

アップグレードパスに無いので最終的にこける

### とりあえず 6.0u3 から ESXi600-201807001 に上げる

6.0u2未満の場合は、まずオフラインバンドルを6.0u3にするところからはじめる

```
# ESXi600-201807001.zip のアップデート...をシミュレーションしてみる
esxcli software vib update --dry-run -d /vmfs/volumes/datastore/ESXi600-201807001.zip

# ESXi600-201807001.zip のアップデート
esxcli software vib update -d /vmfs/volumes/datastore/ESXi600-201807001.zip
# Installation Result
#    Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
#    Reboot Required: true

# メンテナンスモードに切り替える
 esxcli system maintenanceMode get
 esxcli system maintenanceMode set  -e true # 戻しは trueの代わりにfalse
 esxcli system maintenanceMode get

# 再起動 (メンテナンスモードでないと動かない)
esxcli system shutdown reboot -r ESX600-201807001
```

### 6.0u3から7.0へ上げる

注意: 6.0 → 7.0 はそもそもアップグレードパスがないので失敗する

```
# vib install をシミュレーションしなおす
esxcli software vib install --dry-run -d /vmfs/volumes/datastore/VMware-ESXi-7.0U3d-19482537-depot.zip
# Installation Result
#    Message: Dryrun only, host not changed. The following installers will be applied: [BootBankInstaller, LockerInstaller]
#    Reboot Required: true

# vib install しなおす
esxcli software vib install -d /vmfs/volumes/datastore/VMware-ESXi-7.0U3d-19482537-depot.zip
# Installation Result
#    Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
#    Reboot Required: true

# 再起動 (メンテナンスモードでないと動かない)
esxcli system shutdown reboot -r ESX600-201807001

esxcli system version get

# メンテナンスモード戻し
 esxcli system maintenanceMode get
 esxcli system maintenanceMode set  -e false
 esxcli system maintenanceMode get
```

## 6.0u3から ESXi600-201807001 を経由して 6.7u3に上げる

実は6.0u3 →6.7u3b はESX600-201807001 を充てなくても本来はイケるが

特定のベンダのオフラインバンドルではコケるので、あえて当ててある

```
# dry-run
esxcli software vib install --dry-run -d /vmfs/volumes/datastore/ESXi670-201912001.zip
# Installation Result
#    Message: Dryrun only, host not changed. The following installers will be applied: [BootBankInstaller, LockerInstaller]
#    Reboot Required: true
   
# 実行
esxcli software vib install -d /vmfs/volumes/datastore/ESXi670-201912001.zip
# Installation Result
#    Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
#    Reboot Required: true

esxcli system maintenanceMode set  -e true # 戻しは trueの代わりにfalse
esxcli system shutdown reboot -r ESX670
esxcli system version get
esxcli system version get
#    Product: VMware ESXi
#    Version: 6.7.0
#    Build: Releasebuild-15160138
#    Update: 3
#    Patch: 89

esxcli software vib update -d /vmfs/volumes/datastore/ESXi670-202201001.zip
```


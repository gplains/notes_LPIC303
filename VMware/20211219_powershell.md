---
title:VMware PowerCLIのインストールと使用
---
# VMware PowerCLIのインストールと使用

定期的に記憶から消えてしまうので備忘

## 必要なもの

- PowerCLI
  ```
  # あらかじめ管理者権限でPowerShellを実行して
  Install-Module vmware.powercli -AllowClobber
  
  # 実行ポリシーを「制限」から「リモート署名」に変更
  Get-ExecutionPolicy
  Set-ExecutionPolicy remotesigned

  # PowerCLI実行時に証明書の警告が出るときのおまじない
  Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Scope AllUsers
  ```
- vCenter Server
  適当な手段で作る

## 接続～切断

```
 $vc=Connect-VIServer -server (vCenterのIP) -user (任意ユーザ) -Pass (対応するパスワード)
 Disconnect-VIServer $vc -Confirm:$false
```

## 単純にアラーム一覧

```
$state=(Get-Datacenter).ExtensionData.triggeredalarmstate
$state | Select-Object @{name="machine";expression={get-vm -id $_.entity} }, `
 @{name="alarm";expression={Get-AlarmDefinition -id $_.alarm }}
```

## 繰り返してみる

```
while($true){
clear
 $state=(Get-Datacenter).ExtensionData.triggeredalarmstate
 $state | Select-Object @{name="machine";expression={get-vm -id $_.entity} }, `
  @{name="alarm";expression={Get-AlarmDefinition -id $_.alarm }} | ft
 $state=$null
 $state # 何も表示されない
 sleep 4
}
```

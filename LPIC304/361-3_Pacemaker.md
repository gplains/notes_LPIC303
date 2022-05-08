---
title: Pacemakerの導入
---
# Pacemakerの導入

KVM+Vagrantでパッケージ導入したうえでPacemakerを有効化してみる

## Vagrantfile

KVMで内部ブリッジ br0が存在する体で、2ノード作る

- node1 10.0.0.11
- node2 10.0.0.12

```
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.define "node1" do |machine|
    machine.vm.network "public_network", ip: "10.0.0.11" ,
      :dev => "br0",
      :mode => "bridge",
      :type => "bridge"
    machine.vm.hostname = "node1"
    machine.vm.provision "shell", inline: <<-SHELL
    SHELL
  end

  config.vm.define "node2" do |machine|
    machine.vm.network "public_network", ip: "10.0.0.12" ,
      :dev => "br0",
      :mode => "bridge",
      :type => "bridge"
    machine.vm.hostname = "node2"
    machine.vm.provision "shell", inline: <<-SHELL
    SHELL
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo yum update -y        # ベースライン
    # sudo yum install -y httpd # バックエンドサーバ用httpd
    # sudo yum install -y ipvsadm keepalived # LVS用ipvsadm keepalived
    # sudo yum install -y haproxy # LVS用 haproxy
    # sudo yum install -y pacemaker pcs # pacemaker/corosync

    # sudo yum -y install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
    # sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    # sudo yum -y update
    # sudo yum install -y drbd84-utils kmod-drbd84
    SHELL

end
```

## 実際

- 事前準備(node1/node2両方で実施)
  
  ```
  # パッケージアップデートとhttpd/pcsdプロセスの起動
  sudo yum install -y httpd ; sudo systemctl start httpd 
  sudo yum install -y pacemaker pcs;sudo  systemctl start pcsd
  # firewalldのポート開放:KVMのVagrantでは不要
  sudo firewall-cmd --add-service=high-availability --permanent
  sudo firewall-cmd --reload
  # 各ノードでパスワードを定義
  sudo passwd hacluster
  ```

- pcsを使って構築(以降node1)
  
  ここではIPアドレスで認証してますが、ちゃんと双方の/etc/hosts で名前解決できるようにしましょう..
  ```
  # haclusterのID/PWで相互認証
  sudo pcs cluster auth 10.0.0.11 10.0.0.12
  # /etc/corosync/corosync.conf を自動生成できる
  sudo pcs cluster setup --name ha_cluster 10.0.0.11 10.0.0.12
  # sudo pcs cluster setup --name ha_cluster node1 address=10.0.0.11  node2 address=10.0.0.12
  cat /etc/corosync/corosync.conf
  sudo pcs cluster enable --all
  sudo pcs cluster start --all
  sudo pcs status cluster
  sudo pcs status corosync
  # 各種フラグの設定
  sudo pcs property set stonith-enabled=false
  sudo pcs property set no-quorum-policy=ignore
  sudo pcs property set default-resource-stickiness="INFINITY"
  sudo pcs property list
  # VirtualOPとして10.0.0.20 を設定する
  # curl http://10.0.0.20 をノックするとhttp://10.0.0.11 の結果が返る
  sudo pcs resource create Virtual_IP ocf:heartbeat:IPaddr2 ip=10.0.0.20  cidr_netmask=24 op monitor interval=30s
  sudo pcs status corosync
  # 実際に切ったり揚げたりする
  sudo pcs standby node1
  sudo pcs unstandby node1
  ```

## ディストリごとの注意事項

- Ubuntu 
  
  特になし  sudo apt でさくっと入る

- CentOS

  特段問題なし、sudo yum とか sudo dnf でさくっと入る

  ```
  # CentOS 8 Streamのとき
  sudo dnf repolist all
  sudo dnf install --enablerepo=ha pacemaker corosync
  ```

- RHEL8

  パッケージが別
  ```
  subscription-manager repos --enable=rhel-8-for-x86_64-highavailability-rpms
  sudo dnf install pcs
  ```

  別途 High Availability Add-on の契約が必要 <-ここ重要








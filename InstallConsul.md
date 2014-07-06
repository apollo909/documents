<!--
************************************************************
Consulインストール
参照元: http://www.consul.io/intro/getting-started/install.html
Copyright (c) Takehiko OGASAWARA 2014 All Rights Reserved.
************************************************************
-->

# Consulインストール

### 準備
バックアップディレクトリ作成
```
mkdir -p /root/MAINTENANCE/`date "+%Y%m%d"`/{bak,new}
BAK=/root/MAINTENANCE/`date "+%Y%m%d"`/bak
```

必要パッケージの準備
```
apt-get -y install unzip
```

### Consulインストール
```
cd /usr/local/src
sudo wget https://dl.bintray.com/mitchellh/consul/0.3.0_linux_amd64.zip
sudo unzip 0.3.0_linux_amd64.zip
sudo cp -p consul /usr/local/bin/
sudo -u ogalush consul --help
~~~★ヘルプが表示されればOK
```

### 設定
consulは、server/clientの2モードがある。
server: 3〜5台程度推奨
```
consul agent -server -bootstrap -bind=0.0.0.0 -dc=groupA -client=0.0.0.0 -syslog -pid-file=/var/run/consul.pid -data-dir /tmp/consul
```
※ role, dc keysで識別しているので、同じクラスタにする場合はあわせる

### node追加
```
$ consul join 10.0.0.49
Successfully joined cluster by contacting 1 nodes.
$ consul join 10.0.0.48
Successfully joined cluster by contacting 1 nodes.
$ consul join 10.0.0.47
Successfully joined cluster by contacting 1 nodes.
ogalush@serv1:~$ consul members
Node   Address         Status  Type    Build  Protocol
serv1  10.0.0.47:8301  alive   server  0.3.0  2
serv3  10.0.0.49:8301  alive   server  0.3.0  2
serv2  10.0.0.48:8301  alive   server  0.3.0  2
~~~★追加されていればOK
```

試しに落としてみる
```
ogalush@serv1:~$ consul members
Node   Address         Status  Type    Build  Protocol
serv1  10.0.0.47:8301  alive   server  0.3.0  2
serv3  10.0.0.49:8301  left    server  0.3.0  2
~~~★落としたときはleftとなる
serv2  10.0.0.48:8301  alive   server  0.3.0  2
ogalush@serv1:~$ 
```
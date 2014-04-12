<!--
************************************************************
Hadoopインストール手順
参照元: http://www.kkaneko.com/rinkou/cloudcomputing/hadoopinstall.html
        http://codesfusion.blogspot.jp/2013/10/setup-hadoop-2x-220-on-ubuntu.html?m=1
Copyright (c) Takehiko OGASAWARA 2014 All Rights Reserved.
************************************************************
-->

# ノードをインストールする準備

## 準備
バックアップディレクトリ作成
```
$ sudo mkdir -p /root/MAINTENANCE/`date "+%Y%m%d"`/{bak,new}
$ BAK=/root/MAINTENANCE/`date "+%Y%m%d"`/bak
```

## Java DOWNLOAD
```
 ※ブラウザでダウンロードする
 http://www.oracle.com/technetwork/java/javase/downloads/server-jre7-downloads-1931105.html
 → SCPにて転送を行う。
```

## Java 展開
```
$ cd /usr/local/src
$ sudo tar xvzf server-jre-7u51-linux-x64.tar.gz 
$ sudo chown -R root:root jdk1.7.0_51
$ sudo mkdir /usr/lib/jvm
$ sudo mv jdk1.7.0_51 /usr/lib/jvm/.
$ ln -s /usr/lib/jvm/jdk1.7.0_51 /usr/lib/jvm/jdk
$ ls -al /usr/lib/jvm/jdk/bin/java
$ ogalush@name-node:/usr/local/src$ /usr/lib/jvm/jdk/bin/java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
ogalush@name-node:/usr/local/src$ 
```

## Hadoopユーザを入れる
```
$ sudo adduser hduser
 →パスワードは適当に入れる。
```

## sshkey 作成
```
$ sudo su - hduser
$ ssh-keygen -t rsa -P ""
 →パスワードは空で。（ノーパスログインを期待している）
$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
$ ssh hduser@localhost
 → ノーパスでログインできること
```

## ホスト名解決
```
# sudo echo '10.5.5.11 name-node' >> /etc/hosts
# sudo echo '10.5.5.12 data-node' >> /etc/hosts
```

## 環境変数設定
```
$ vi ~/.bashrc
----
#Hadoop variables
export JAVA_HOME=/usr/lib/jvm/jdk
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_CONF_DIR=$HADOOP_INSTALL/etc/hadoop
----

$ source ~/.bashrc
$ echo $JAVA_HOME
```

# Hadoopインストール
## ダウンロード
```
$ sudo su -
# cd /usr/local/src
# wget http://ftp.meisei-u.ac.jp/mirror/apache/dist/hadoop/common/hadoop-2.3.0/hadoop-2.3.0.tar.gz
# tar -xvzf /usr/local/src/hadoop-2.3.0.tar.gz
# mv hadoop-2.3.0 /usr/local/.
# ln -s /usr/local/hadoop-2.3.0 /usr/local/hadoop
# ls -al /usr/local
# chown -R hduser:hduser /usr/local/hadoop/
```

## データストア向けディレクトリ作成
```
# sudo mkdir -p /usr/local/hadoop-datastore; chown hduser:hduser /usr/local/hadoop-datastore
```

## 初期設定
```
```

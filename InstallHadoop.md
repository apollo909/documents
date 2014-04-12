<!--
************************************************************
Hadoopインストール手順
参照元: http://www.kkaneko.com/rinkou/cloudcomputing/hadoopinstall.html
        http://codesfusion.blogspot.jp/2013/10/setup-hadoop-2x-220-on-ubuntu.html?m=1
Copyright (c) Takehiko OGASAWARA 2014 All Rights Reserved.
************************************************************
-->

# hadoopインストール手順（まずはhdfsを使用できるように)

## 構成
```
namenode:  name-node(10.5.5.11)
datanode:  data-node(10.5.5.12, 10.5.5.13)
```

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
$ sudo ln -s /usr/lib/jvm/jdk1.7.0_51 /usr/lib/jvm/jdk
$ ls -al /usr/lib/jvm/jdk/bin/java
$ /usr/lib/jvm/jdk/bin/java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
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
$ ssh hduser@name-node
$ ssh hduser@data-node
$ ssh hduser@data-node2
 → namenode, data-nodeの各ホストで、相互にsshノーパスでログインできること
```

## ホスト名解決
```
# echo '10.5.5.11 name-node' >> /etc/hosts
# echo '10.5.5.12 data-node' >> /etc/hosts
# echo '10.5.5.13 data-node2' >> /etc/hosts
# cat /etc/hosts
```

## 環境変数設定
```
$ vi ~/.bashrc
----
#Hadoop variables
export JAVA_HOME=/usr/lib/jvm/jdk
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_PREFIX=/usr/local/hadoop
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_CONF_DIR=$HADOOP_INSTALL/etc/hadoop
export HADOOP_SECURE_DN_USER=hdfs
----
$ source ~/.bashrc
$ echo $JAVA_HOME

# vi /root/.bashrc
----
#Hadoop variables
export JAVA_HOME=/usr/lib/jvm/jdk
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_PREFIX=/usr/local/hadoop
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_CONF_DIR=$HADOOP_INSTALL/etc/hadoop
export HADOOP_SECURE_DN_USER=hdfs
----
# source ~/.bashrc
# echo $JAVA_HOME
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
→ hadoopのhdfs出力先ディレクトリとして使用する。
  hadoopのデフォルト(core-site.xml:hadoop.tmp.dir)はOS再起同時にデータストアが消えてしまう。
  (/tmp/hadoop〜であるため)
```

## 初期設定(config)
```
# mkdir -p /tmp/hadoop
# chown hduser:hduser /tmp/hadoop
# cd /usr/local/hadoop/etc/
# cp -p /usr/local/hadoop/share/hadoop/hadoop-yarn/hadoop-yarn-common/yarn-default.xml /usr/local/hadoop/etc/hadoop/yarn-site.xml
# cp -p /usr/local/hadoop/share/hadoop/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml /usr/local/hadoop/etc/hadoop/hdfs-site.xml
# cp -p /usr/local/hadoop/share/hadoop/hadoop-project-dist/hadoop-common/core-default.xml /usr/local/hadoop/etc/hadoop/core-site.xml
# cp -p /usr/local/hadoop/share/hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml /usr/local/hadoop/etc/hadoop/mapred-site.xml

# vi /usr/local/hadoop/etc/hadoop/core-site.xml
# vi /usr/local/hadoop/etc/hadoop/yarn-site.xml
# vi /usr/local/hadoop/etc/hadoop/core-site.xml
# vi /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

## 初期設定(namenode)
```
hduser@name-node$ hdfs namenode -format test
```

## daemonの起動
```
hduser@name-node$ hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start namenode
hduser@name-node$ mr-jobhistory-daemon.sh start historyserver --config $HADOOP_CONF_DIR
hduser@data-node$ hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start datanode
```

## daemonの停止
```
hduser@name-node$ hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs stop namenode
hduser@data-node$ hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs stop datanode
```

## その他コマンド
```
(1) hadoopへノード追加後、同期させるコマンド
hduser@[name|data]-node$ hdfs-sites.xml: dfs.hostの指定ファイルへホストを追加
 ※今回の場合は/usr/local/hadoop/etc/hadoop/datanode.conf
hduser@name-node$ hdfs hadoop dfsadmin -refreshNodes

(2) GUI管理画面
 http://name-node(IPアドレス):50070/dfshealth.html
```


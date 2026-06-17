---
title: 大数据管理平台Ambari3.0最新版本二次开发入门实践指南
author: 大数据从业者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1NjYyMDE5OA==&mid=2247486447&idx=1&sn=e33b6b7f3be722e24be285592d97e286&chksm=fac7aeebbd3f4587a11dedd525187c625f90e7e6e003c4672fec4a4141c113f9cfebe66fee33&mpshare=1&scene=24&srcid=0430Jtocqxs9vp6AM1HFurOB&sharer_shareinfo=6ed1cccb27b5074a856c65f93cee9f09&sharer_shareinfo_first=6ed1cccb27b5074a856c65f93cee9f09#rd
---

## 前言

Apache Ambari项目初期旨在通过开发用于配置、管理、监控Hadoop集群的软件，使Hadoop集群管理更加简洁方便。因为Ambari灵活的插件化设计机制，现在已经广泛用于支持更多其他大数据组件的管理，用户可以根据需要自定义组件。Amabri提供直观且便于使用的Web管理界面，也提供RESTful API支持。

数十年来，Ambari已经被很多大数据平台厂商(如Hortonworks)产品集成使用，本文主要介绍Ambari3.0最新版本的源码编译、安装部署、配置管理、组件集成等内容。欢迎关注微信公众号：大数据从业者

## 源码编译

工欲善其事必先利其器，对于二次开发，源码编译作为入门步骤、必不可少。具体的源码编译打包命令如下：(自定义是否打rpm包)

```
mvn -B -T 2C clean install package rpm:rpm -Drat.skip=true -DskipTests -Dmaven.test.skip=true -Dfindbugs.skip=true -Dcheckstyle.skip=true
```

详细报错信息如下：

```
Unable to build the RPM: Error while executing process. Cannot run program "rpmbuild" (in directory "/home/mySourceCode/ambari/target/rpm/ambari/SPECS"): error=2, No such file or directory
```

根本原因：缺少rpmbuild命令

解决方法： 

```
yum search rpm-buildyum install rpm-build
```

查看ambari-server编译打包结果(rpm、tar)：

```
ll ambari-server/target/rpm/ambari-server/RPMS/x86_64ll ambari-server/target/
```

查看ambari-server rpm对应spec文件，可以知道具体的安装操作过程：

```
ll ambari-server/target/rpm/ambari-server/SPECS/
```

 查看ambari-agent编译打包结果(rpm、tar)：

查看ambari-agent rpm对应spec文件，可以知道具体的安装操作过程：

```
ll ambari-agent/target/rpm/ambari-agent/SPECS/
```

## RPM方式部署

设置远端yum repo

```
vim /etc/yum.repos.d/ambari_remote_repo.repo[ambari_remote_repo]name=ambaribaseurl=http://FelixZhnew/ambariRepo/enabled=1gpgcheck=0 
```

设置本地yum repo（非必选，适用于单节点）

```
vim /etc/yum.repos.d/ambari_repo.repo[ambari_local_repo]name=Ambari Local Repositorybaseurl=file:///home/ambariRepoenabled=1gpgcheck=0 
```

制作rpm repo

```
mkdir /home/ambariRepocp ambari-server/target/rpm/ambari-server/RPMS/x86_64/ambari-server-3.0.0.0-0.x86_64.rpm /home/ambariRepo/cp ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ambari-agent-3.0.0.0-0.x86_64.rpm /home/ambariRepo/cd /home/ambariRepo/createrepo . 
```

部署配置httpd暴漏repo(也可以用nginx等)

```
yum install httpdvim /etc/httpd/conf/httpd.conf
```

配置ServerName、DocumentRoot、Directory等内容:

```
systemctl start httpd yum clean allyum repolist
```

 安装部署

```
yum install ambari-serveryum install ambari-agent 
```

 Tar方式部署

注意：适用于研发调试环境

根据ambari-server spec文件梳理整合的ambari-server安装过程详细流程如下：

```
cd ambari-server/target/ambari-server-3.0.0.0.0-distcp -r etc/init /etc/cp etc/init.d/ambari-server /etc/init.d/cp -r etc/ambari-server/ /etc/cp usr/sbin/* /usr/sbin/cp -r usr/lib/ambari-server /usr/libcp -r var/lib/ambari-server /var/lib/sh /var/lib/ambari-server/install-helper.sh install
```

根据ambari-agent spec文件梳理整合的ambari-agent安装过程详细流程如下：

```
cd ambari-agent/target/ambari-agent-3.0.0.0.0cp etc/init/ambari-agent.conf /etc/init/cp etc/init.d/ambari-agent /etc/init.dcp -r etc/ambari-agent/ /etc/cp -r usr/lib/ambari-agent/ /usr/libcp -r var/lib/ambari-agent/ /var/lib//var/lib/ambari-agent/install-helper.sh install
```

## 数据库部署

可以使用mysql或者postgresql两种类型数据库，笔者这里使用mysql。

```
yum install -y python3-psycopg2yum -y install https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpmrpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023yum -y install mysql-serversystemctl start mysqld.servicesystemctl enable mysqld.service
```

修改数据库默认密码：

```
sudo grep 'temporary password' /var/log/mysqld.logALTER USER 'root'@'localhost' IDENTIFIED BY 'FelixZh#2024';FLUSH PRIVILEGES; 
```

创建用户及相关数据库

```
-- Create Ambari user and grant privilegesCREATE USER 'ambari'@'localhost' IDENTIFIED BY 'FelixZh#2024';GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';CREATE USER 'ambari'@'%' IDENTIFIED BY 'FelixZh#2024';GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';-- Create required databasesCREATE DATABASE ambari CHARACTER SET utf8 COLLATE utf8_general_ci;CREATE DATABASE hive;CREATE DATABASE ranger;CREATE DATABASE rangerkms;-- Create service usersCREATE USER 'hive'@'%' IDENTIFIED BY 'FelixZh#2024';GRANT ALL PRIVILEGES ON hive.* TO 'hive'@'%';CREATE USER 'ranger'@'%' IDENTIFIED BY 'FelixZh#2024';GRANT ALL PRIVILEGES ON *.* TO 'ranger'@'%' WITH GRANT OPTION;CREATE USER 'rangerkms'@'%' IDENTIFIED BY 'FelixZh#2024';GRANT ALL PRIVILEGES ON rangerkms.* TO 'rangerkms'@'%';
```

## AmbariServer配置启动

导入ambari schema到上述部署的数据库中

```
mysql -uambari -pFelixZh#2024 ambari < /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql 
```

配置ambari-server

```
mkdir /usr/share/javawget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar -O /usr/share/java/mysql-connector-java.jarambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jarecho "server.jdbc.url=jdbc:mysql://localhost:3306/ambari?useSSL=true&verifyServerCertificate=false&enabledTLSProtocols=TLSv1.2" >> /etc/ambari-server/conf/ambari.propertiesambari-server setup -s \  -j /home/pkg/jdk1.8.0_221/ \  --ambari-java-home /home/pkg/jdk-17.0.2/ \  --database=mysql \  --databasehost=localhost \  --databaseport=3306 \  --databasename=ambari \  --databaseusername=ambari \  --databasepassword=FelixZh#2024
```

启动ambari-sevrer

```
ambari-server start
```

```
http://10.121.198.221:8080/admin/admin
```

## AmbariAgent配置启动

配置agent上报server的主机名

```
vim /etc/ambari-agent/conf/ambari-agent.inihostname=FelixZhnew
```

启动ambari-agent

```
ambari-agent start
```

## Ambari组件集成

目前Ambari3.0版本stacks已经删除HDP组件集成的相关内容，主要是因为HDP被CDH收购之后不再开源了。所以，目前只保留BIGTOP组件集成。BIGTOP组件集成的安装部署操作如下：

```
cd /home/ambariRepowget -r -np -nH --cut-dirs=4 --reject 'index.html*' https://www.apache-ambari.com/dist/bigtop/3.3.0/rocky8/
```

根据Ambari向导安装组件即可，效果演示如下：

BIGTOP适配的组件有限，目前支持列表如下（对于不支持的组件，需要二次开发集成合入）：

注意：本人提供免费的HDP版本资源包，如有需要请自行到公众号大数据从业者发送 hdp 领取下载地址。

## 结束

本文主要介绍Ambari3.0版本源码编译、安装部署、配置管理、组件集成等内容。二次开发主要集中在stacks中服务包内容和对应服务的RPM包修改。后续更新二次开发相关内容。欢迎关注大数据从业者公众号！
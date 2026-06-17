---
title: 博文推荐｜Apache Doris 单节点 Docker 集群制作教程
author: ApacheDoris
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5MDEyODc1OA==&mid=2247508077&idx=1&sn=e76c9e5eeb4b946e90a4703b3dfbb871&chksm=cfe3b074f89439628abdb9040b638ae1418a4e5dc2d77f07bff5a61ba67468a45dd499226e41&mpshare=1&scene=24&srcid=0310MiQI4VQ1QEa8gUFxY3sQ&sharer_sharetime=1646905873992&sharer_shareid=4145ee29355a80b1b2b01921ded70e3a#rd
---

**前言**

Apache Doris 是当下非常流行的 MPP 架构 OLAP 数据库，很多同学想自学/测试 Doris 的使用和能力，但是又苦于没有环境或者畏惧冗长的编译+搭建过程，整个过程极大的劝退了很多有些尝试意愿、但又由于各种客观因素无法承担过高学习成本导致尝试失败的同学，故此我们向社区贡献了三个不同设计版本的制作及安装方式并提供下载地址，以此降低大家的学习门槛和提升学习/测试效率。

**重要说明：该教程提供的编译方式及运行环境都以单节点部署伪集群为目标，如想体验完整 Apache Doris 数据库的能力，请以完整集群部署，单节点伪集群仅适用于学习、功能测试所用！**

**版本说明**

### 01    极速体验版

#### **1.1. 优点**

1.超快速的部署体验（网速 OK 的话十五分钟部署完毕）

2.单节点部署

3.支持多环境运行：虚拟机/云服务器/支持 Docker 的物理机（Mac/Win/Linux）

#### **1.2. 缺点**

    1.数据存储是在 Docker 容器中，如容器如损坏，会导致数据丢失

    2.若非干净纯净的系统环境，可能需要手动执行部分 BE 注册 FE 的命令

#### **1.3. 适用人群**

学生、培训机构、体验/测试人员

#### **1.4. 安装建议**

系统为纯净新系统最佳，无需任何修改即可开箱即用

### 02    完全部署版

#### **2.1. 优点**

1. 完整的环境部署（ MySQL-Client 等组件）
2. 自由的部署安排（有众多可选安装参数）
3. 无惧 Docker 容器损坏（最小降低损失，可极速恢复）
4. 单节点部署
5. 支持多环境运行：虚拟机/云服务器，暂未适配物理机（后续升级版本会支持）

#### **2.2. 缺点**

1. 安装过程时间较长（视网速和机器性能而定）
2. 安装步骤多，代表可能故障率较高

#### **2.3. 适用人群**

学生、培训机构、体验/测试人员中的持续性教学受众（数据不易丢失）

#### **2.4 安装建议**

该版本建议完完全全的纯净新系统，以此降低安装故障率

### 03    存算分离版

该版本还在制作过程中，教程及相关文档后续推出，可视为完全部署版的 Plus 版本。

**目的**

该教程最后成果模块提供了各个版本下载地址，只需在服务器拉取不同版本shell脚本运行即可，在/opt/docker/doris/sbin目录下会有 start\_doris\_docker.sh和stop\_doris\_docker.sh 脚本支持一键启停，同时会在一键部署的过程中将两个脚本添加至环境变量，最大程度简化单节点测试部署和启停操作。

**步骤过程可以忽略**，除非有定制化的一键部署 Docker 集群的镜像集群制作需求，大可不必照着教程再来一遍，官方已提供了下载地址，无需重复劳动。

**环境**

**01    环境一**

* 服务器：腾讯云 2C 4G 6M 一台
* OS：CentOS 7.6
* Docker-V：20.10.12
* Doris-V：1.0 beta
* MySQL-Client-V：5.7
* FE-Num：1
* BE-Num：3

### **02    环境二**

* 服务器：Win虚拟机 8C 44G 一台
* OS：CentOS 7.6
* Docker-V：20.10.12
* Doris-V：1.0 beta
* MySQL-Client-V：5.7
* FE-Num：1
* BE-Num：5

**步骤**

01    安装 Docker 环境

1.Docker 要求 CentOS 系统的内核版本高于 3.10 ，首先查看系统内核版本是否满足

```
uname -r
```

2.使用 root 权限登录系统，确保 yum 包更新到最新

```
sudo yum update -y
```

3.假如安装过旧版本，先卸载旧版本

```
sudo yum remove docker  docker-common docker-selinux docker-engine
```

4.安装需要的软件包， yum-util 提供 yum-config-manager 功能，另外两个是 devicemapper 驱动依赖的

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

5.设置 yum 源（加速 yum 下载速度）

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

6.查看所有仓库中所有 Docker 版本，并选择特定版本安装，一般可直接安装最新版

```
yum list docker-ce --showduplicates | sort -r
```

7.安装 Docker

* 安装最新稳定版本

```
sudo yum install docker-ce -y  #安装的是最新稳定版本，因为repo中默认只开启stable仓库
```

* 安装指定版本

```
sudo yum install <FQPN> -y# 例如：sudo yum install docker-ce-20.10.11.ce -y
```

8. 启动并加入开机启动

```
sudo systemctl start docker #启动dockersudo systemctl enable docker #加入开机自启动
```

9. 查看 Version，验证是否安装成功

```
docker version
```

   若出现 Client 和 Server 两部分内容，则证明安装成功

02    容器创建及测试

**在创建之前，请准备好已完成编译的 FE/BE 文件，此教程不再赘述编译过程。**

1.拉取 Doris 编译镜像做测试

```
```
# 拉取docker pull apache/incubator-doris:build-env-ldb-toolchain-latest
```
```

2.创建 Doris-Docker 的文件（包括元数据文件夹）

```
```
mkdir -p /opt/docker/doris
```
```

3.将编译好的 FE 和 BE 拷贝至 Docker 文件群内

```
```
cp -r 编译好的Doris根目录/fe/ /opt/docker/doris/cp -r 编译好的Doris根目录/be/ /opt/docker/doris/be-01cp -r 编译好的Doris根目录/be/ /opt/docker/doris/be-02cp -r 编译好的Doris根目录/be/ /opt/docker/doris/be-03
```
```

4.启动 FE-Docker

```
```
docker run -it -p 8030:8030 -p 9030:9030 -d --name=doris-fe -v /opt/docker/doris/fe:/opt/doris/fe -v /opt/docker/doris/doris-meta:/opt/doris/doris-meta apache/incubator-doris:build-env-ldb-toolchain-latest
```
```

5.进入 FE-Docker 以及安装组件

```
```
# 进入fe-dockerdocker exec -ti doris-fe /bin/bash# 安装net-tools用于查看IPyum install net-tools -y
```
```

6.修改FE配置

```
```
# 查看fe-docker的IPv4地址ifconfig# 修改配置文件vim /opt/doris/fe/conf/fe.conf# 取消priority_networks的注解，并根据Docker的网段进行配置priority_networks = 172.17.0.0/16 #这里要根据你Docker的IP确定
```
```

7.切换 Docker-JDK 版本

```
```
# 切换Java版本为JDK1.8，该镜像默认为JDK11alternatives --set java java-1.8.0-openjdk.x86_64alternatives --set javac java-1.8.0-openjdk.x86_64export JAVA_HOME=/usr/lib/jvm/java-1.8.0# 校验是否切换版本成功java -version
```
```

8.配置 FE-Docker 的环境变量

```
```
# 配置环境变量vim /etc/profile.d/doris.shexport DORIS_HOME=/opt/doris/fe/export PATH=$PATH:$DORIS_HOME/bin# 保存并sourcesource /etc/profile.d/doris.sh
```
```

9.启动 Doris-FE

```
```
start_fe.sh --daemon
```
```

10.检查 FE 是否启动成功

* 检查是否启动成功，JPS 命令下有没有 Palo-Fe 进程
* FE 进程启动后，会首先加载元数据，根据 FE 角色的不同，在日志中会看到 transfer from UNKNOWN to MASTER/FOLLOWER/OBSERVER。最终会看到 thrift server started 日志，并且可以通过 MySQL 客户端连接到 FE，则表示 FE 启动成功。
* 也可以通过如下连接查看是否启动成功：   
  http://fe\_host:fe\_http\_port/api/bootstrap  
  如果返回： {"status":"OK","msg":"Success"}  
  则表示启动成功，其余情况，则可能存在问题。
* 外网环境访问 http://fe\_host:fe\_http\_port  查看是否可以访问 WebUI 界面，登录账号默认为 root，密码为空

注：如果在 fe.log 中查看不到启动失败的信息，也许在 fe.out 中可以看到。

11.宿主机安装 MySQL 客户端

```
```
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.37-1.el7.x86_64.rpm-bundle.tartar -xvf mysql-5.7.37-1.el7.x86_64.rpm-bundle.tarrpm -ivh mysql-community-common-5.7.37-1.el7.x86_64.rpmrpm -ivh mysql-community-libs-5.7.37-1.el7.x86_64.rpmrpm -ivh mysql-community-client-5.7.37-1.el7.x86_64.rpm
```
```

12.连接 FE 并修改密码

```
```
mysql -h FE-Docer的IP -P 9030 -urootSET PASSWORD FOR 'root' = PASSWORD('your_password');# 也可以创建新用户CREATE USER 'test' IDENTIFIED BY 'test_passwd';
```
```

后续链接时需要使用如下格式

```
```
mysql -h FE_HOST -P9030 -uusername -ppassword
```
```

添加 BE 节点注册信息

```
```
ALTER SYSTEM ADD BACKEND "host:port";# 其中 host 为 BE 所在节点 ip；port 为 be/conf/be.conf 中的 heartbeat_service_port，默认9050。
```
```

13.启动 BE-Docker

```
```
docker run -it -p 9061:9060 -d --name=doris-be-01 -v /opt/docker/doris/be-01:/opt/doris/be apache/incubator-doris:build-env-ldb-toolchain-latestdocker run -it -p 9062:9060 -d --name=doris-be-02 -v /opt/docker/doris/be-02:/opt/doris/be apache/incubator-doris:build-env-ldb-toolchain-latestdocker run -it -p 9063:9060 -d --name=doris-be-03 -v /opt/docker/doris/be-03:/opt/doris/be apache/incubator-doris:build-env-ldb-toolchain-latest
```
```

14.进入 BE-Docker 以及安装组件

```
```
# 进入fe-docker,以01为例docker exec -ti doris-be-01 /bin/bash# 安装net-tools用于查看IPyum install net-tools -y
```
```

15.修改 BE 配置

```
```
# 查看fe-docker的IPv4地址ifconfig# 修改配置文件vim /opt/doris/be/conf/be.conf# 取消priority_networks的注解，并根据Docker的网段进行配置priority_networks = 172.17.0.0/16 #这里要根据你Docker的IP确定
```
```

16.配置 BE-Docker 的环境变量

```
```
# 配置环境变量vim /etc/profile.d/doris.shexport DORIS_HOME=/opt/doris/be/export PATH=$PATH:$DORIS_HOME/bin# 保存并sourcesource /etc/profile.d/doris.sh
```
```

17.启动 Doris-BE

```
```
start_be.sh --daemon
```
```

18.检查 BE 是否启动成功

* BE 进程启动后，如果之前有数据，则可能有数分钟不等的数据索引加载时间。
* 如果是 BE 的第一次启动，或者该 BE 尚未加入任何集群，则 BE 日志会定期滚动 waiting to receive first heartbeat from frontend 字样。表示 BE 还未通过 FE 的心跳收到 Master 的地址，正在被动等待。这种错误日志，在 FE 中 ADD BACKEND 并发送心跳后，就会消失。如果在接到心跳后，又重复出现 master client, get client from cache failed.host: , port: 0, code: 7 字样，说明 FE 成功连接了 BE，但 BE 无法主动连接 FE。可能需要检查 BE 到 FE 的 rpc\_port 的连通性。
* 如果 BE 已经被加入集群，日志中应该每隔 5 秒滚动来自 FE 的心跳日志：get heartbeat, host: xx.xx.xx.xx, port: 9020, cluster id: xxxxxx，表示心跳正常。
* 其次，日志中应该每隔 10 秒滚动 finish report task success. return code: 0 的字样，表示 BE 向 FE 的通信正常。
* 同时，如果有数据查询，应该能看到不停滚动的日志，并且有 execute time is xxx 日志，表示 BE 启动成功，并且查询正常。
* 也可以通过如下连接查看是否启动成功：   
  http://be\_host:be\_http\_port/api/health  
  如果返回： {"status": "OK","msg": "To Be Added"}  
  则表示启动成功，其余情况，则可能存在问题。

注：如果在 be.INFO 中查看不到启动失败的信息，也许在 be.out 中可以看到。

19.测试连通性

```
```
# 登录FE-MySQLmysql -h FE_HOST -P9030 -uusername -ppassword# 执行命令查看BE运行情况。如一切正常，isAlive 列应为 true。SHOW PROC '/backends';
```
```

20.若连通性测试成功，则循环完成其他 BE 节点的部署即可

### 

### 03    安装 ETCD 环境（若多节点 Dokcer 需配置|单节点可忽略）

1.配置 Hosts 文件映射

```
```
vim /etc/hosts你本机内网IP地址 master
```
```

2.安装 ETCD

```
```
# 安装ETCDyum install -y etcd# 重启ETCDsystemctl restart etcd
```

3.设置开机启动
```

```
```
systemctl enable etcd
```

4.修改 ETCD 配置
```

```
```
# 先查找本机的IP地址 ifconfig# 备份原始配置文件 cp /etc/etcd/etcd.conf /etc/etcd/etcd.conf.bak# 编辑ETCD的conf文件 vim /etc/etcd/etcd.conf# 修改监听客户端地址为ETCD_LISTEN_CLIENT_URLS="http://master:2379,http://127.0.0.1:2379,http://master:4001,http://127.0.0.1:4001"# 修改通知客户端地址为ETCD_ADVERTISE_CLIENT_URLS="http://master:2379,http://master:4001"# 保存退出
```

5.设置 ETCD 网段
```

```
```
# Flannel使用Etcd进行配置，来保证多个Flannel实例之间的配置一致性，所以需要在etcd上进行如下配置（'/atomic.io/network/config'这个key与上文/etc/sysconfig/flannel中的配置项FLANNEL_ETCD_PREFIX是相对应的，错误的话启动就会出错）etcdctl mk /atomic.io/network/config '{"Network":"172.20.0.0/16","SubnetMin":"172.20.1.0","SubnetMax":"172.20.254.0"}'
```
```

6.重启 ETCD

```
```
systemctl restart etcd
```

7.测试
```

```
# 查看ETCD进程是否存在ps -ef|grep etcd# 查看端口使用情况，因为ETCD默认TCP:2379端口通讯lsof -i:2379# 使用get命令查看是否设置成功etcdctl get /atomic.io/network/config # 若出现以下信息，则代表设置成功{"Network":"172.20.0.0/16","SubnetMin":"172.20.1.0","SubnetMax":"172.20.254.0"}# 查看cluster-healthetcdctl -C http://master:4001 cluster-healthetcdctl -C http://master:2379 cluster-health# 若出现如下信息，则代表成功member 8e9e05c52164694d is healthy: got healthy result from http://你IP地址:2379（和4001）
```

04    安装 Flannel 环境（若多节点 Dokcer 需配置，单节点可忽略）

1. Yum 安装 Flannel

```
```
yum install -y flannel
```
```

2.配置 Flannel

```
```
# 备份原始配置文件cp /etc/sysconfig/flanneld /etc/sysconfig/flanneld.bak# 编辑配置文件vim /etc/sysconfig/flanneld# 修改以下配置项FLANNEL_ETCD_ENDPOINTS="http://master:2379"
```

3.设置开机自启
```

```
systemctl enable flanneld.service
```

4.启动 Flannel

```
```
systemctl start flanneld.service
```

5.重启 Docker
```

```
systemctl restart docker
```

6.测试

```
# 查看Flannel进程ps -ef | grep flannel
```

05    测试及远程连接

可使用 Navicat 等远端工具连接FE，地址为部署了 FE 服务的单机外网 IP，端口为 9030，如图所示

### 

### 

### 

### 

### 06    SHELL 脚本设计及开发

#### **6.1. 完整部署版整体设计示意图**

#### **6.2. 思路梳理**

##### **6.2.1 极速体验版（极速体验免除安装）**

1.默认 1FE 3BE 安装

2. Docker 安装（可参照步骤1）

3.拉取 Docker 镜像群

```
```
docker pull freeoneplus/doris-fe:latestdocker pull freeoneplus/doris-be:latest
```

4.创建 FE-Docker 容器
```

```
docker run -it -p 8030:8030 -p 9030:9030 -d --name=doris-fe freeoneplus/doris-fe:1.0
```

```
5.进入 FE-Docker 并获取 IPv4 地址
```

```
docker exec -it doris-fe /bin/bashifconfigexitdocker exec -d doris-fe /bin/bash /opt/doris/fe/start_fe.sh --daemon
```

```
6.循环创建 BE-Docker 容器并启动 BE
```

```
docker run -it -p 9061:9060 -d --name=doris-be-01 freeoneplus/doris-be:1.0docker exec -d doris-be-01 /bin/bash /opt/doris/be/start_be.sh --daemondocker run -it -p 9062:9060 -d --name=doris-be-02 freeoneplus/doris-be:1.0docker exec -d doris-be-02 /bin/bash /opt/doris/be/start_be.sh --daemondocker run -it -p 9063:9060 -d --name=doris-be-03 freeoneplus/doris-be:1.0docker exec -d doris-be-03 /bin/bash /opt/doris/be/start_be.sh --daemon
```

```
7.提示用户进行BE注册

> ## 亲爱的用户，欢迎使用 Apache Doris-极简版-Docker集群！ 接下来的文字请认真阅读： 1. 此版本集群为极简版单节点docker集群，所有数据均挂载在Docker集群内，请谨慎修改或删除容器！ 2. 此版本预制注册三个BE节点至FE，但可能由于不同环境影响，预先注册的IP地址可能会出现错误，所以请仔细观察FE的预制IP地址：${FE-IP地址}，若以上地址为172.17.0.2，则无需做任何修改即可直接使用，如果是其他数值，则需要进行链接FE进行BE注册 3. 您可以使用任意MySQL-Client或者MySQL工具连接FE-MySQL-Server 若宿主机（您的虚拟机/云服务器）有MySQL-Client，则需要执行以下命令链接FE-MySQL-Server mysql -h ${FE-IP地址} -P 9030 -uroot -p123456 若您使用外网机器链接FE-MySQL-Server，则需要填入以下参数，您需要提前打开9030外网端口 url：您的服务器外网IP（虚拟机则视网络桥接方式） port：9030 username：root password：123456 然后执行以下命令清除已注册至FE的BE节点信息
>
>
>
> ## 以下需要逻辑处理 预设的三个BE地址为[172.17.0.3,172.17.0.4,172.17.0.5] 该地址应为FE-IP地址最后一位自增3，所以如果预设错误，需要给出删除语句和增添语句 比如FE-IP为 172.17.0.4，则需要给出删除[172.17.0.3,172.17.0.4]两个BE节点的语句 ALTER SYSTEM DECOMMISSION BACKEND "${FE-IP地址}:9050"; 然后再给出新增的两个节点的IP[172.17.0.6,172.17.0.7]注册语句 ALTER SYSTEM ADD BACKEND "${FE-IP地址}:9050"; 以上需要逻辑处理 感谢您的安装和使用Apache Doris！ 感谢您为开源世界作出的一份贡献！ 如有问题请关注ApacheDoris微信公众号回复“加群”进入社区交流群获取答疑~
```

##### **6.2.2 完整部署版（数据落盘无惧丢失）**

1.校验脚本执行口令，防止误操作

* 输出一段文字说明
* 等待接收 Doris 这五个字母，成功则继续，未成功则终止

2.依次询问参数配置设置，接收参数，可参考的有：

* 是否默认配置安装（Y/N）
* BE 数量（默认为 3）
* root 密码（默认为空）
* 操作员账户名称（默认无）
* 操作员账户密码（默认无）
* FE-Http-Port 端口（默认 8030）
* FE-MySQL-Cli-Port 端口（默认 9030）

3.创建宿主机资源目录并进入

```
```
mkdir -p /opt/docker/doris/cd /opt/docker/doris/
```
```

4.拉取编译好的文件包至上述目录（当前版本为 Apache Doris-1.0.0 bate 测试版）

```
wget https://jiafeng2022.oss-cn-beijing.aliyuncs.com/doris-1.0.0-jdk8-20220301.tar.gz
```

5.解压文件包

```
tar -zxvf /opt/docker/doris/apache-doris-install.tar.gz
```

6.根据传参的 BE 数量循环复制 BE 目录，以默认数量为样例，命令执行为

```
```
cp -r /opt/docker/doris/be /opt/docker/doris/be-01cp -r /opt/docker/doris/be /opt/docker/doris/be-02cp -r /opt/docker/doris/be /opt/docker/doris/be-03
```

7.监测 Docker 是否安装
```

```
docker version
```

8.如果已安装则跳过，未安装则安装 Docker

```
```
# 监测内核版本，若小于3.10则终止安装并通知失败，告知失败原因uname -r# 如果大于3.10则开始安装，依次执行以下命令sudo yum update -ysudo yum install -y yum-utils device-mapper-persistent-data lvm2sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.reposudo yum install docker-ce -ysudo systemctl start docker sudo systemctl enable docker # 执行结束，监测执行是否都已成功docker version
```

9.监测 MySQL-Client 是否已安装
```

```
```
mysql --version
```

10.如果已安装则跳过，未安装则安装 MySQL-Client
```

```
```
mkdir -p /opt/softwarecd /opt/softwarewget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.37-1.el7.x86_64.rpm-bundle.tartar -xvf mysql-5.7.37-1.el7.x86_64.rpm-bundle.tarrpm -ivh mysql-community-common-5.7.37-1.el7.x86_64.rpmrpm -ivh mysql-community-libs-5.7.37-1.el7.x86_64.rpmrpm -ivh mysql-community-client-5.7.37-1.el7.x86_64.rpm
```

11.拉取 Doris 编译镜像为基础环境镜像
```

```
docker pull apache/incubator-doris:build-env-ldb-toolchain-latest
```

12.制作 FE 容器

* 构建FE容器

```
docker run -it -p 8030:8030 -p 9030:9030 -d --name=doris-fe -v /opt/docker/doris/fe:/opt/doris/fe -v /opt/docker/doris/doris-meta:/opt/doris/doris-meta apache/incubator-doris:build-env-ldb-toolchain-latest
```

```
* 进入容器
```

```
```
docker exec -ti doris-fe /bin/bash
```

* 修改 FE 配置文件
```

```
vim /opt/doris/fe/conf/fe.conf# 如FE两个对外端口都是默认值，则无需修改，若有改变，则改变该值http_port = 8030query_port = 9030# 修改网段priority_networks = 172.17.0.0/16
```

```
* 切换 JDK 版本
```

```
# 切换Java版本为JDK1.8，该镜像默认为JDK11alternatives --set java java-1.8.0-openjdk.x86_64alternatives --set javac java-1.8.0-openjdk.x86_64export JAVA_HOME=/usr/lib/jvm/java-1.8.0
```

```
* 配置 Doris 环境变量
```

```
# 配置环境变量vim /etc/profile.d/doris.shexport DORIS_HOME=/opt/doris/fe/export PATH=$PATH:$DORIS_HOME/bin# 保存并sourcesource /etc/profile.d/doris.sh
```

```
* 安装 Net-Tools 工具以便于查看 IP 地址
```

```
yum install net-tools -y
```

```
* 使用命令查看该 Docker 的 IPv4 地址，并记录下来
```

```
```
ifconfig
```
```

* 启动 FE

```
start_fe.sh# 最好执行命令以后再等待10秒左右
```

```
* 退出该容器，返回宿主机
```

```
```
exit
```

13.用 MySQL-Client 连接 Doris
```

```
mysql -h ${记录下的FE-Docker的IPv4地址} -P ${默认9030，如有改变则使用改变后的query-port} -uroot
```

14.注册 BE 至 FE

```
ALTER SYSTEM ADD BACKEND "${FE-Docker的IPv4地址的第四位自增1}:9050";# 这里需要说明的是，这命令执行时应该是根据BE的数量来循环的，比如BE为默认值3，记录下FE-Docker的地址为172.17.0.3，那么就应该循环添加 172.17.0.4:9050、172.17.0.5:9050、172.17.0.6:9050三条注册信息，以此类推
```

15.若有用户修改密码和注册了操作员账户，则执行以下命令

```
# 修改密码SET PASSWORD FOR 'root' = PASSWORD('${填写的root密码}');# 也可以创建新用户CREATE USER '${填写的操作员账户}' IDENTIFIED BY '${填写的操作员密码}';
```

16.退出 MySQL-Client

```
```
exit
```
```

17.制作 BE 容器，该处应该进入以 BE 数量为最大数值从 1 开始的循环中（以 BE-01 为例）

**假设 BE 的节点数量从 1 自增的变量为 n，在以下示例中取值方式为 ${n}**

* 构建 BE 容器

```
# 标准格式为如下所示，其中三处被替换为${n}docker run -it -p 906${n}:9060 -d --name=doris-be-0${n} -v /opt/docker/doris/be-0${n}:/opt/doris/be apache/incubator-doris:build-env-ldb-toolchain-latest# 示例docker run -it -p 9061:9060 -d --name=doris-be-01 -v /opt/docker/doris/be-01:/opt/doris/be apache/incubator-doris:build-env-ldb-toolchain-latest
```

```
* 进入容器
```

```
# 这里需要注意，也是要根据循环进行取值docker exec -ti doris-be-0${n} /bin/bash
```

```
* 修改 BE 配置文件
```

```
vim /opt/doris/be/conf/be.conf# 取消priority_networks的注解，并根据Docker的网段进行配置priority_networks = 172.17.0.0/16 #这里要根据你Docker的IP确定
```

```
* 配置 BE 环境变量
```

```
# 配置环境变量vim /etc/profile.d/doris.shexport DORIS_HOME=/opt/doris/be/export PATH=$PATH:$DORIS_HOME/bin# 保存并sourcesource /etc/profile.d/doris.sh
```

```
* 启动 BE
```

```
start_be.sh
```

```
* 退出容器，开始下一次循环
```

```
```
exit
```

18.循环结束，清除临时解压缩及部分下载文件
```

```
rm -rf /opt/software/*.rpmrm -rf /opt/docker/doris/apache-doris-install.tar.gz
```

19.制作启动、停止脚本（前提 Docker 容器是启动的，若未启动则报错）

启动脚本需以 start\_doris\_docker.sh 命名，停止脚本以 stop\_doris\_docker.sh 命名

两个脚本均写在 /opt/docker/doris/sbin/ 目录下

* 创建目录

```
mkdir -p /opt/docker/doris/sbin/
```

```
* 启动脚本内容

>> 启动 FE
```

```
docker exec -d doris-fe /bin/bash /opt/doris/fe/start_fe.sh --daemon
```

```
>> 循环启动 BE

```
docker exec -d doris-be-0${n} /bin/bash /opt/doris/be/start_be.sh --daemon
```

* 停止脚本内容
```

```
    >> 循环停止 BE

```
docker exec -d doris-be-0${n} /bin/bash /opt/doris/be/stop_be.sh --daemon
```

         >>停止 FE
```

```
docker exec -d doris-fe /bin/bash /opt/doris/fe/stop_fe.sh --daemon
```

* 配置环境变量

```
```
vim /etc/profile.d/doris-docker.shexport DORIS_DOCKER_HOME=/opt/docker/doris/sbinexport PATH=$PATH:$DORIS_DOCKER_HOME
```
```

* 刷新环境变量

```
```
source /etc/profile.d/doris-docker.sh
```
```

**成果**

**极速体验版部署流程**

（此脚本部署将部署最新版本 Apache Doris）

```
```
wget http://download.freeoneplus.com/doris_docker_fast_install.shsh ./doris_docker_fast_install.sh
```
```

**完全部署版部署流程**

（此脚本部署将部署最新版本 Apache Doris ）

```
```
wget http://download.freeoneplus.com/doris_docker_whole_install.shsh ./doris_docker_whole_install.sh
```

假设需要指定版本的部署，请使用以下部署流程

```
# 极速体验版部署流程wget http://download.freeoneplus.com/doris_docker_fast_install_${指定版本号}.shsh ./doris_docker_fast_install_${指定版本号}.sh# 案例：极速体验版 Apache Doris 0.15版本wget http://download.freeoneplus.com/doris_docker_fast_install_0.15.shsh ./doris_docker_fast_install_0.15.sh  
# 完全部署版部署流程wget http://download.freeoneplus.com/doris_docker_whole_install_${指定版本号}.shsh ./doris_docker_whole_install_${指定版本号}.sh# 案例：完全部署版 Apache Doris 0.15版本wget http://download.freeoneplus.com/doris_docker_whole_install_0.15.shsh ./doris_docker_whole_install_0.15.sh
```

当前支持版本对照表

| Apache Doris version | 是否支持 |
| --- | --- |
| 1.0.0-beta | 支持 |
| 0.15 | 3月12日起支持 |
| 0.14及以下 | 不支持 |
```

**测试**

**使用官网的 SSB 测试集进行测试**

**单节点规模：**

    CPU：8C

    内存：44G

    硬盘：400G

    FE：1

    BE：5

| 脚本名称 | 查询时间（ms） |
| --- | --- |
| q1.1 | 926ms |
| q1.2 | 461ms |
| q1.3 | 410ms |
| q2.1 | 13383ms |
| q2.2 | 12001ms |
| q2.3 | 11354ms |

**- 作者介绍 -**

集群制作 Author：苏奕嘉  
脚本研发 Author：种   益  
调研测试 Author：杨春东

欢迎关注：

Apache Doris(incubating)官方公众号

**【精彩文章】**

> [社区人物志｜缪翎：见证开源世界的女性力量](http://mp.weixin.qq.com/s?__biz=Mzg5MDEyODc1OA==&mid=2247507958&idx=1&sn=c518856a910d751faf25264eb8939b43&chksm=cfe3cfeff89446f9a224afd4933aa7e116828b2bb196018a4522ebb87aac35c27bee6ac1209f&scene=21#wechat_redirect)
>
> [应用实践 | Apache Doris 在小米集团的运维实践](http://mp.weixin.qq.com/s?__biz=Mzg5MDEyODc1OA==&mid=2247507687&idx=1&sn=641dc49e697931c9d49cda2fb57bd495&chksm=cfe3cefef89447e8e8d65c3808a32702646977451791d288e580343572262e46a3956090bf3d&scene=21#wechat_redirect)
>
> [从NoSQL到Lakehouse，Apache Doris的13年技术演进之路](http://mp.weixin.qq.com/s?__biz=Mzg5MDEyODc1OA==&mid=2247500759&idx=1&sn=04968e36335a6efe5c9bc395b9b6d2b5&chksm=cfe3d3cef8945ad85d6cd7abdeb891419d286f0b3c32679cb05308f60e3fcfeabc26cb9b9bc0&scene=21#wechat_redirect)

相关链接：

**Apache Doris官方网站：**

http://doris.incubator.apache.org

**Apache Doris Githu****b：**

https://github.com/apache/incubator-doris

**Apache Doris 开发者邮件组：**

dev@doris.apache.org
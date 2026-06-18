> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070302_安全权限/070302_核心知识点/安全权限入口与访问控制边界|安全权限入口与访问控制边界]]、[[07_工程与架构/0703_工程实践与质量保障/070302_安全权限/070302_知识地图|070302_安全权限知识地图]]

---
title: 京东一面：如何用 Nginx 禁止国外 IP 访问网站！
author: 架构师社区
date:
url: http://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247542295&idx=2&sn=eb6cc0e08cf34d1144c3f2f47fee959e&chksm=fbb1afe9ccc626ff6cf9e0cbe5e617940b4ba496e5fa991999d732d0b6494a779a896b4bb709&mpshare=1&scene=24&srcid=0625y4nTgndmVqR72mdeGrLy&sharer_sharetime=1656113279114&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

来源：toutiao.com/i6860736292339057156/

之前看了下 Nginx 的访问日志，发现每天有好多国外的 IP 地址来访问我的网站，并且访问的内容基本上都是恶意的。因此我决定禁止国外 IP 来访问我的网站。

想要实现这个功能有很多方法，下面我就来介绍基于 Nginx 的 ngx\_http\_geoip2 模块来禁止国外 IP 访问网站。

**# 安装 geoip2 扩展依赖**

```
[root@fxkj ~]# yum install libmaxminddb-devel -y
```

**# 下载 ngx\_http\_geoip2\_module 模块**

```
[root@fxkj tmp]#  git clone https://github.com/leev/ngx_http_geoip2_module.git        [ro tmp]#
```

解压模块到指定路径

我这里解压到 /usr/local 目录下：

```
[root@fxkj tmp]# mv ngx_http_geoip2_module/ /usr/local/        [root@fxkj local]# ll ngx_http_geoip2_module/        total 60        -rw-r--r-- 1 root root  1199 Aug 13 17:20 config        -rw-r--r-- 1 root root  1311 Aug 13 17:20 LICENSE        -rw-r--r-- 1 root root 23525 Aug 13 17:20 ngx_http_geoip2_module.c        -rw-r--r-- 1 root root 21029 Aug 13 17:20 ngx_stream_geoip2_module.c        -rw-r--r-- 1 root root  3640 Aug 13 17:20 README.md
```

**# 安装 nginx 模块**

首先说明下环境，我的 nginx 版本是 1.16，在网上查了下安装 ngx\_http\_geoip2 模块至少需要 1.18 版本及以上，因此此次安装我是升级 nginx1.18，添加 ngx\_http\_geoip2 模块。

下载 nginx 1.18 版本：

```
[root@fxkj ~]# yum install libmaxminddb-devel -y
```

解压 nginx1.18 软件包，并升级为 nginx1.18，添加 ngx\_http\_geoip2 模块。

需要注意：

* 升级 nginx，添加 nginx 模块，只需要编译，然后 make。不需要 make instll，不然线上的 nginx 会被新版本 nginx 完完整整的替换掉。
* 编译前需要看下 nginx 当前安装了哪些模块。

```
[root@fxkj tmp]# /usr/local/nginx/sbin/nginx -V        nginx version: nginx/1.16.0        built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)        built with OpenSSL 1.0.2k-fips 26 Jan 2017        TLS SNI support enabled        configure arguments: –with-http_stub_status_module –prefix=/usr/local/nginx –user=nginx –group=nginx –with-http_ssl_module –with-stream
```

编译安装：

```
[root@fxkj tmp]# tar -xf nginx-1.18.0.tar.gz        [root@fxkj tmp]# cd nginx-1.18.0/        [root@fxkj nginx-1.18.0]# ./configure --with-http_stub_status_module \        --prefix=/usr/local/nginx \        --user=nginx --group=nginx --with-http_ssl_module --with-stream \        --add-module=/usr/local/ngx_http_geoip2_module        [root@fxkj nginx-1.18.0]# make        [root@fxkj nginx-1.18.0]# cp /usr/loca/nginx/sbin/nginx /usr/loca/nginx/sbin/nginx1.16    #备份        [root@fxkj nginx-1.18.0]# cp objs/nginx /usr/local/nginx/sbin/    #用新的去覆盖旧的        [root@fxkj nginx-1.18.0]# pkill nginx     #杀死nginx        [root@fxkj nginx-1.18.0]# /usr/local/nginx/sbin/nginx    #再次启动Nginx
```

查看 nginx 版本，以及安装的模块：

```
[root@fxkj nginx-1.18.0]# /usr/local/nginx/sbin/nginx -V        nginx version: nginx/1.18.0        built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)        built with OpenSSL 1.0.2k-fips 26 Jan 2017        TLS SNI support enabled        configure arguments: –with-http_stub_status_module –prefix=/usr/local/nginx –user=nginx –group=nginx –with-http_ssl_module –with-stream –add-module=/usr/local/ngx_http_geoip2_module
```

**# 下载最新的 IP 地址数据库文件**

模块安装成功后，还要在 Nginx 里指定数据库，在安装运行库时默认安装了两个，位于 /usr/share/GeoIP/ 目录下，一个只有 IPv4，一个包含 IPv4 和 IPv6。

登录 www.maxmind.com 网址，创建账户，下载最新的库文件。（账户创建就不演示了）点击左侧，Download Files：

选择 GeoLite2 Country，点击 Download GZIP 下载即可：

上传到 /usr/share/GeoIP/ 下并解压：

```
[root@fxkj local]# cd /usr/share/GeoIP/        [root@fxkj GeoIP]# ll        total 69612        lrwxrwxrwx. 1 root root       17 Mar  7  2019 GeoIP.dat -> GeoIP-initial.dat        -rw-r--r--. 1 root root  1242574 Oct 30  2018 GeoIP-initial.dat        lrwxrwxrwx. 1 root root       19 Mar  7  2019 GeoIPv6.dat -> GeoIPv6-initial.dat        -rw-r--r--. 1 root root  2322773 Oct 30  2018 GeoIPv6-initial.dat        -rw-r--r--  1 root root  3981623 Aug 12 02:37 GeoLite2-Country.mmdb
```

**# 配置 nginx 配置文件**

修改前先备份配置文件：

```
[root@fxkj ~]# cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf-bak        [root@fxkj ~]# vim /usr/local/nginx/conf/nginx.conf
```

在 http 中添加几行，定义数据库文件位置：

```
geoip2 /usr/share/GeoIP/GeoLite2-City.mmdb {        auto_reload 5m;        $geoip2_data_country_code country iso_code;        }        map $geoip2_data_country_code $allowed_country {default yes;        CN no;        }
```

在 server 中的 location 下添加条件，如果满足 IP 是国外 IP，就执行下面的 return 动作，我这里定义了 3 种，注释了其中两个。

当访问 IP 是国外 IP，直接返回 404：

```
if ($allowed_country = yes) {        # return https://www.baidu.com;        # return /home/japan;        return 404;        }
```

修改完毕后，检测下配置文件，重新加载下 nginx：

```
[root@fxkj ~]# /usr/local/nginx/sbin/nginx -t        nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok        nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful        [roo@fxkj ~]# /usr/local/nginx/sbin/nginx -s reload
```

**# 模拟测试验证**

使用海外节点的服务器去访问网站，这里我的 IP 是来自于韩国：

可以看到访问网站报错 404 Not Found：

我们再来看下 nginx 的访问日志：

```
“13.125.1.194 – – [14/Aug/2020:16:15:51 +0800] “GET /favicon.ico HTTP/1.1” 404 548 “https://www.fxkjnj.com/” “Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,like Gecko) Chrome/84.0.4147.125 Safari/537.36”
```

至此，我们通过 Nginx 来实现禁止国外 IP 访问网站就结束了~
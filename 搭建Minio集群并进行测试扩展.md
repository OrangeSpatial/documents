# 搭建Minio集群并进行测试扩展



## 准备

四个节点

192.168.30.40

192.168.30.41

192.168.30.42

192.168.30.43

前两个一组，后两个一组，在前两个的基础上扩充后两个

每个节点预设4个磁盘

export1-4

## 基础环境

centos8 4g 500G

![image-20201007171513800](搭建Minio集群并进行测试扩展.assets/image-20201007171513800.png)

安装wget，vim

~~~bash
yum install vim wget -y
~~~

在`/home`下创建export1-4目录

~~~bash
mkdir -p /home/{data1,data2,data3,data4}
~~~

![image-20201007172151224](搭建Minio集群并进行测试扩展.assets/image-20201007172151224.png)

修改源

CentOS 8**

```cmd
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
yum clean all
yum makecache
yum update
```

网络配置

~~~bash
vim /etc/sysconfig/network-scripts

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens160"
UUID="05dec958-c3a8-4528-b428-fa27d90816b5"
DEVICE="ens160"
ONBOOT="yes"
IPADDR="192.168.30.40"
PREFIX="24"
GATEWAY="192.168.30.254"
DNS1="*.*.*.*"
IPV6_PRIVACY="no"

~~~

## 修改主机名及hosts

~~~
hostnamectl set-hostname minio40
~~~

```
vim /etc/hosts
192.168.30.40 minio40
192.168.30.41 minio41
192.168.30.42 minio42
192.168.30.43 minio43

```





## 安装NTP

minio40 运行

~~~bash

# yum install chrony
# vim /etc/chrony.conf

server ntp1.aliyun.com iburst
server time4.aliyun.com iburst
server 0.centos.pool.ntp.org iburst   //设置ntp服务器
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
allow 192.168.30.0/24

# systemctl enable chronyd.service
# systemctl start chronyd.service
~~~

其他节点只需添加

```
server minio40 iburst
```

验证

```
chronyc sources

```



## 安装minio

创建脚本目录

```bash
mkdir -p /opt/minio
```

下载

~~~
cd /opt/minio
wget https://dl.min.io/server/minio/release/linux-amd64/minio
wget http://dl.minio.org.cn/server/minio/release/linux-amd64/minio
chmod +x minio
~~~

编写启动脚本

~~~
vim /opt/minio/run.sh

#!/bin/bash
export MINIO_ACCESS_KEY=Minio
export MINIO_SECRET_KEY=admin@1234
/opt/minio/minio server --config-dir /etc/minio \
http://minio4{0...1}/home/export{1...4}

chmod +x /opt/minio/run.sh
~~~

编写服务脚本

~~~cmd
vim /usr/lib/systemd/system/miniod.service

[Unit]
Description=Minio service
Documentation=https://docs.minio.io/
 
[Service]
WorkingDirectory=/opt/minio/
ExecStart=/opt/minio/run.sh
 
Restart=on-failure
RestartSec=5
 
[Install]
WantedBy=multi-user.target
~~~

服务重载并启动

~~~cmd
systemctl daemon-reload
systemctl start miniod
systemctl enable miniod
~~~

关闭防火墙

~~~cmd
systemctl stop firewalld
# 开通 2380、2379、9000端口
firewall-cmd --zone=public --add-port=2380/tcp --add-port=2379/tcp --add-port=9000/tcp --permanent && firewall-cmd --reload
firewall-cmd --list-ports
systemctl restart firewalld

~~~

### 测试

# 一定关闭防火墙或开启端口

minio40/9000，minio40/9000

## 横向扩展

修改脚本

```
vim /opt/minio/run.sh

#!/bin/bash
export MINIO_ACCESS_KEY=Minio
export MINIO_SECRET_KEY=admin@1234
/opt/minio/minio server --config-dir /etc/minio \
http://minio4{0...1}/home/export{1...4} http://minio4{2...3}/home/export{1...4}
```



## nginx

~~~conf
upstream minio {
       server 192.168.30.40:9000 weight=10 max_fails=2 fail_timeout=30s;
       server 192.168.30.41:9000 weight=10 max_fails=2 fail_timeout=30s;
       server 192.168.30.42:9000 weight=10 max_fails=2 fail_timeout=30s;
       server 192.168.30.43:9000 weight=10 max_fails=2 fail_timeout=30s;
    }
   server {
    	listen 9000;
        server_name localhost;
        charset utf-8;
        default_type text/html;
        location /{
           proxy_set_header Host $http_host;
           proxy_set_header X-Forwarded-For $remote_addr;
           client_body_buffer_size 10M;
           client_max_body_size 10G;
           proxy_buffers 1024 4k;
           proxy_read_timeout 300;
           proxy_next_upstream error timeout http_404;
           proxy_pass http://minio;
        }
     }
~~~




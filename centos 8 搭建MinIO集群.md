# centos 8 搭建MinIO集群

## 安装系统

虚拟机安装 4cpu 8G内存 1T存储

### 准备基础环境

### 1. 备份

```cmd
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

### 2. 下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

**CentOS 6**

```cmd
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
```

或者

```cmd
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
```

**CentOS 7**

```cmd
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

或者

```cmd
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

**CentOS 8**

```cmd
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

或者

```cmd
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

### 3. 清除缓存

~~~cmd
yum clean all
~~~

### 4. 缓存数据

~~~cmd
yum makecache
~~~

### 5. 更新数据

~~~cmd
yum update
~~~

### 6. 修改Ip

~~~cmd
vim /etc/sysconfig/network-scripts
~~~



~~~cmd
# 重启
nmcli c reload
~~~



## 安装MinIo

[参考](https://www.cnblogs.com/rongfengliang/p/9197315.html)

搭建方式： 两个节点 192.168.30.95  192.168.30.96 ，各挂载两个磁盘 /home/data1, /home/data2

### 1.查看磁盘情况

~~~cmd
df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             3.8G     0  3.8G   0% /dev
tmpfs                3.9G     0  3.9G   0% /dev/shm
tmpfs                3.9G  8.6M  3.9G   1% /run
tmpfs                3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/cl-root   50G  2.0G   48G   4% /
/dev/mapper/cl-home  965G  6.8G  958G   1% /home
/dev/sda2            976M  179M  731M  20% /boot
/dev/sda1            599M  6.9M  592M   2% /boot/efi
tmpfs                781M     0  781M   0% /run/user/0
~~~

### 2.创建磁盘目录(两个节点各创建两个)

~~~cmd
mkdir -p /home/{data1,data2}

~~~

### 3.创建脚本目录

~~~cmd
mkdir -p /opt/minio
~~~

### 4.下载 minio

~~~
cd /opt/minio
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
~~~

### 5.编写启动脚本

~~~
vim /opt/minio/run.sh

#!/bin/bash
export MINIO_ACCESS_KEY=Minio
export MINIO_SECRET_KEY=admin@1234
/opt/minio/minio server --config-dir /etc/minio \
http://192.168.30.95/home/data1 http://192.168.30.95/home/data2 \
http://192.168.30.96/home/data1 http://192.168.30.96/home/data2 \

chmod +x /opt/minio/run.sh
~~~

### 6. 编写服务脚本

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

### 7.服务重载并启动

~~~cmd
systemctl daemon-reload
systemctl start miniod
systemctl enable miniod
~~~

### 8.关闭防火墙

~~~cmd
systemctl stop firewalld
# 开通 2380、2379、9000端口
firewall-cmd --zone=public --add-port=2380/tcp --add-port=2379/tcp --add-port=9000/tcp --permanent && firewall-cmd --reload
firewall-cmd --list-ports
systemctl restart firewalld

~~~

### 9.测试

http://192.168.30.95:9000  或  http://192.168.30.96:9000

## nginx代理访问

此处使用docker 部署nginx

[官方文档](https://docs.min.io/docs/setup-nginx-proxy-with-minio.html)

### 1.编辑conf.d/default.conf

~~~conf
   
 upstream welearn_minio {
     server 192.168.30.95:9000 weight=5 ;
     server 192.168.30.96:9000 weight=5 ;
 }

server {
    listen       80;
    server_name  localhost;
    # 不能超过2048
    client_max_body_size 2048m;
    charset utf-8;
    add_header 'Access-Control-Allow-Origin' '*';   

    location / {
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header Host $http_host;

       proxy_connect_timeout 300;
       # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
       proxy_http_version 1.1;
       proxy_set_header Connection "";
       chunked_transfer_encoding off;

       proxy_pass http://welearn_minio; 
       # If you are using docker-compose this would be the hostname i.e. minio
       # Health Check endpoint might go here. See https://www.nginx.com/resources/wiki/modules/healthcheck/
       # /minio/health/live;
     }
     
     # Proxy requests to the bucket "photos" to MinIO server running on port 9000
     location /photos/ {
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header Host $http_host;

       proxy_connect_timeout 300;
       # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
       proxy_http_version 1.1;
       proxy_set_header Connection "";
       chunked_transfer_encoding off;

       proxy_pass http://welearn_minio;
     }
}

~~~



## 扩容

将扩容后的节点添加至启动文件run.sh中

~~~sh
#!/bin/bash
export MINIO_ACCESS_KEY=Minio
export MINIO_SECRET_KEY=admin@1234
/opt/minio/minio server --config-dir /etc/minio \
	http://192.168.30.95/home/data1 http://192.168.30.95/home/data2 \
	http://192.168.30.96/home/data1 http://192.168.30.96/home/data2 \
	http://192.168.30.97/home/data3 http://192.168.30.97/home/data4 \
	http://192.168.30.98/home/data3 http://192.168.30.98/home/data4 

~~~

重启每个节点



~~~terminal
Unable to read 'format.json' from http://192.168.30.98:9000/home/data4: Expected 'storage' API version 'v20', instead found 'a4', please upgrade the servers
~~~

升级服务

安装客户端

下载客户端

```bash
./mc config host add minio1 http://192.168.30.95 Minio admin@1234 --api s3v4
./mc config host add minio2 http://192.168.30.96 Minio admin@1234 --api s3v4
./mc config host add minio3 http://192.168.30.97 Minio admin@1234 --api s3v4
./mc config host add minio4 http://192.168.30.98 Minio admin@1234 --api s3v4
```

更新

~~~bash
./mc admin update
~~~


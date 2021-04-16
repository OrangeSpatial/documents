# Docker安装使用

## 1.安装环境

系统：centos 7

## 2.安装

官方地址：https://docs.docker.com/install/linux/docker-ce/centos/

切换至root用户下，

su

password：

执行以下命令：

```
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

启用夜间库（可选）

```
yum-config-manager --enable docker-ce-nightly
yum-config-manager --enable docker-ce-test
yum-config-manager --disable docker-ce-nightly
```

安装Docker 引擎 社区版

```
#安装最新版
yum install docker-ce docker-ce-cli containerd.io

#查询版本
yum list docker-ce --showduplicates | sort -r

#安装具体指定版本
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

docker --version
```

启动docker

```
systemctl start docker
```

运行hello-world

```
docker run hello-world
```

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:4df8ca8a7e309c256d60d7971ea14c27672fc0d10c5f303856d7bc48f8cc17ff
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:

安装成功！！
```



## 3.aliyun加速

~~~ json
docker@default-online:~$ cat ~/.docker/config.json
{
    "auths": {
        "registry.cn-hangzhou.aliyuncs.com": {
            "auth": "XXXXXXXXXXXXXXXXXXXXXX"
        }
    }
}
~~~




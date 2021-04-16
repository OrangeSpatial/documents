# OpenStack 初探

网址：https://docs.openstack.org/devstack/latest/

测试环境：

- Ubuntu 18.04 (Bionic Beaver)
- 8g内存
- 50G存储

### 添加stack用户

```
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
```

赋予sudo 权限

```
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo su - stack
```

### 下载DevStack

```
$ sudo apt install git
$ git clone https://opendev.org/openstack/devstack
$ cd devstack
```

### 创建local.conf文件

~~~
$ sudo apt install vim
$ vim local.conf
~~~

local.conf文件内容，四个密码都设置为123456

```
[[local|localrc]]
ADMIN_PASSWORD=123456
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```

### 开始安装

~~~ 
$ ./stack.sh
~~~


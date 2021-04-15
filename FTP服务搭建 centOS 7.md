FTP服务搭建 centOS 7



修改源：

1，登陆root帐号

2，cd /etc/yum.repos.d

3，cp CentOS-Base.repo CentOS-Base.repo.bak
4，wget http://mirrors.aliyun.com/repo/Centos-7.repo
5，mv Centos-7.repo CentOS-Base.repo

清除缓存

6， yum clean all

缓存数据

7， yum makecache

更新数据

8，yum update



默认情况下，vsftpd 服务支持本地用户（/etc/passwd）登录  

查看所有用户

```
cat /etc/passwd #查看所有用户信息
cat /etc/group  #查看用户组
cat /etc/shadow  #查看用户状态

cut -d: -f 1 /etc/passwd #查看用户

cut -d: -f 1 /etc/group #查看用户组
```

/sbin/nologin 系统用户

/bin/bash 普通用户

管理员



创建用户

~~~
普通用户
useradd username
不能登录的用户
useradd -g groupname -M -d /homeDir -s /sbin/nologin username

useradd -M -s /sbin/nologin username
useradd -M -s /bin/false username
useradd -M -s /usr/sbin/nologin username

选项：
  -b, --base-dir BASE_DIR       新账户的主目录的基目录
  -c, --comment COMMENT         新账户的 GECOS 字段
  -d, --home-dir HOME_DIR       新账户的主目录
  -D, --defaults                显示或更改默认的 useradd 配置
 -e, --expiredate EXPIRE_DATE  新账户的过期日期
  -f, --inactive INACTIVE       新账户的密码不活动期
  -g, --gid GROUP               新账户主组的名称或 ID
  -G, --groups GROUPS   新账户的附加组列表
  -h, --help                    显示此帮助信息并推出
  -k, --skel SKEL_DIR   使用此目录作为骨架目录
  -K, --key KEY=VALUE           不使用 /etc/login.defs 中的默认值
  -l, --no-log-init     不要将此用户添加到最近登录和登录失败数据库
  -m, --create-home     创建用户的主目录
  -M, --no-create-home          不创建用户的主目录
  -N, --no-user-group   不创建同名的组
  -o, --non-unique              允许使用重复的 UID 创建用户
  -p, --password PASSWORD               加密后的新账户密码
  -r, --system                  创建一个系统账户
  -R, --root CHROOT_DIR         chroot 到的目录
  -P, --prefix PREFIX_DIR       prefix directory where are located the /etc/* files
  -s, --shell SHELL             新账户的登录 shell
  -u, --uid UID                 新账户的用户 ID
  -U, --user-group              创建与用户同名的组
  -Z, --selinux-user SEUSER             为 SELinux 用户映射使用指定 SEUSER

~~~



给用户添加密码

```
passwd username
echo 123|passwd --stdin username
```



查看用户是否存在

~~~
id username
~~~



### 新建FTP用户

```
useradd -d /data/ftp -g ftp -s /sbin/nologin ftpuser2
```

-d:指定用户登录时的起始目录
 -g:用户组
 -s /sbin/nologin指定用户只能用于ftp登录，拒绝用户登录系统

修改该FTP用户密码
 `passwd ftpuser`

目录权限设置

~~~
chown -R ftp:ftpuser2 /data/ftp

usermod -G groupname username           #给username用户设置groupname附属用户组
usermod -a -G groupname username        #给username用户添加至groupname附属用户组

#给用户设置添加多个用户组
usermod -g main_groupname -G groupname1,groupname2 username     #给main_groupname用户设置主用户组web组，groupname1,groupname2附属用户组
gpasswd -a username groupname                #给username用户设置groupname用户组
~~~



### 创建chroot_list文件，添加用户

```
vim /etc/vsftpd/chroot_list
```



匿名模式下，可以使用普通用户登录访问ftp服务

win资源管理器 ftp://192.168.10.10

右击窗体空白处，点击登录即可登录到普通用户下

普通用户登录后可以上传和下载文件，与此同时还可以修改文件

在linux下链接ftp服务器，需要安装ftp或lftp

ftp下可以使用

~~~
？ //查看帮助
put  filename //上传文件
get  filename //下载文件
~~~

匿名用户无法上传文件，可以下载文件

在匿名用户模式下切换至普通用户

~~~
user username
~~~

以上普通用户为ftp服务端的本地用户



修改配置文件vsftpd.conf

1. 关闭防火墙和selinux
2. 配置yum源
3. 软件安装
4. 确定安装成功
5. 查看安装列表
6. 修改配置文件完成指定任务
7. 启动服务，开启自启动（ss、netstat、lsof -i :port）
8. 测试验证



防火墙关闭

~~~
systemctl stop firewalld.service 

systemctl disable firewalld.service #禁止开机启动
~~~

开放端口：(开放端口后需要重启防火墙才可以)

~~~
firewall-cmd --zone=public --add-port=21/tcp --permanent
firewall-cmd --zone=public --add-port=20/tcp --permanent

#查询端口是否开启
firewall-cmd --query-port=21/tcp

#重启防火墙
firewall-cmd --reload

#查询开启的端口
firewall-cmd --list-port

开放服务：
firewall-cmd --permanent --zone=public --add-service=ftp
firewall-cmd --add-service=ftp --permanent
firewall-cmd --complete-reload
~~~

selinux

~~~
#状态查看

sestatus

getenforce


临时关闭

setenforce 0

永久关闭

vim /etc/sysconfig/selinux

修改权限775 还原660

修改SELINUX=disabled

重启

reboot


~~~

或者进行一下操作：

getsebool -a | grep ftp

setsebool -P ftpd_full_access on



 /etc/pam.d/vsftpd 修改 

修改/etc/pam.d/vsftpd
将auth required pam_shells.so修改为->auth required pam_nologin.so 即可 



完成以下需求：

1. 必须使用账户密码登录来下载文件
2. 不允许匿名登录
3. 指定用户登录访问目录
4. 用户登录后只能在指定目录下活动

**1. 在ftp服务端创建用户给密码**

~~~
创建用户名ftpuser，密码为123

useradd  ftpuser
echo 123|passwd --stdin ftpuser
~~~

**2.禁止匿名用户访问**
修改配置文件
进入配置文件目录

~~~
cd /etc/vsftpd
~~~

备份配置文件

~~~
cp vsftpd.conf vsftpd.conf.bak
vim vsftpd.conf
~~~

**3.关闭匿名用户修改**

如果看不懂文档可以执行以下命令查看

~~~
man 5 vsftpd.conf
~~~

搜索关键词

~~~
/keyword
#禁匿名修改vsftpd.conf
anonymous_enable=NO
~~~

重启服务，测试

~~~
service vsftpd restart
systemctl restart vsftpd.service
~~~



**4.指定访问目录**

~~~
#创建访问目录
mkdir /data/ftpresources -p
#创建文件：
touch filename

#修改配置文件vsftpd.conf
#文末添加
local_root=/data/ftpresources
~~~

*pwd 查看目录路径*

4.在指定目录下活动

~~~
#修改配置文件vsftpd.conf
chroot_local_user=YES
#打开上传功能：
allow_writeable_chroot=YES
~~~

设置开机启动

```
systemctl enable vsftpd.service
```





### 问题：

1. 访问时出现 “打开ftp服务器上的文件夹时发生错误，请检查是否有权限访问该文件夹”错误 

    **解决办法：**设置IE浏览器>>Internet选项>>高级>>将“使用被动FTP（用于防火墙和DSL调制解调器的兼容）”选项去掉>>确定即可 
    
2. win文件管理器打开报错

   

![image-20191124125940385](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124125940385.png)



​          浏览器可以下载，因此是系统问题
































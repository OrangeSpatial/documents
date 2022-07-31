# Ubuntu18.04 配制Samba服务

https://www.jianshu.com/p/c4579605a737

### 更新当前软件

~~~
sudo apt-get update
~~~

### 安装samba服务

~~~
sudo apt-get install samba samba-common
~~~

### 创建共享目录

创建用户目录下的dl，用于共享

~~~
mkdir ~/dl
~~~

### 添加用户

~~~
# 添加用户(cheng，设置对应密码)。 
sudo smbpasswd -a learner
~~~





### 配置samba的配置文件

sudo vim /etc/samba/smb.conf 

共享目录 */home/cheng/dl*

用户名：cheng

~~~
[share]
comment = share folder
browseable = yes
path = /home/cheng/dl
create mask = 0700
directory mask = 0700
valid users = cheng
force user = cheng
force group = cheng
public = yes
available = yes
writable = yes
~~~

### 重启服务

~~~cmd
sudo service smbd restart
或
sudo systemctl restart smbd
~~~

### 验证

局域网内windows系统地址栏输入

~~~
\\ip
~~~

输入账号密码后会拒绝访问等

![smb1](.\.\img\smb1.png)

--》在运行中输入“gpedit.msc”来启动本地组策略编辑器。

--》在组策略编辑器中找到“计算机配置”-->网络-->Lanman工作站

![smb1](.\.\img\smb2.png)

--》已启用

![smb1](.\.\img\smb3.png)



### java 访问

maven 引入

~~~
<!-- https://mvnrepository.com/artifact/jcifs/jcifs -->
        <dependency>
            <groupId>jcifs</groupId>
            <artifactId>jcifs</artifactId>
            <version>1.3.17</version>
        </dependency>
~~~

注意：	根路径不是dl，是share，windows下为实际名称

~~~
"smb://192.168.30.60/share/"
~~~


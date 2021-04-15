



# FTP服务使用说明

进入任何一台虚拟机，如果资料已在自己的虚拟机上可以直接上传

步骤为：（以192.168.30.30为例）

## 第一种方式：windows平台

打开任何一个windows 的资源管理器，输入ftp://192.168.30.80，出现登录界面

![image-20191124162105911](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124162105911.png)

<img src="C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124162140441.png" alt="image-20191124162140441" style="zoom:67%;" />

输入账号密码进行登录。

### 账号密码说明：

目前建立了四个ftp服务账号：

密码都为：dl@user

一个读写账号：

~~~
dlUser
~~~

三个只读账号：

~~~
dlUser1
dlUser2
dlUser3
~~~



**用读写账号dlUser登录：**

<img src="C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124162600435.png" alt="image-20191124162600435" style="zoom: 67%;" />

目前根目录下就一个文件夹  /dl

<img src="C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124162622179.png" alt="image-20191124162622179" style="zoom: 80%;" />

右键---新建--文件夹，可以新建文件夹，但不能直接新建文件，文件需要在客户端（当前登录电脑）新建之后上传

![image-20191124163019524](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124163019524.png)

上传文件只需将客户机上文件拖进你需要上传到的文件夹内即可，也可以删除上传错误的文件，但不能将一个在服务器上的文件剪贴（拖拽）到其他文件夹中，只能下载到本机，再删除原文件，再上传到指定文件夹中。

![image-20191124163633562](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124163633562.png)

**用只读账号dlUser1/2/3登录：**

切换账号只需，右击--登录，输入对应的账户信息即可

此处用dlUser1登录

![image-20191124164001554](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164001554.png)

**只能将文件复制到客户机，不能新建文件夹，不能上传文件，不能删除文件**

下载文件成功！！

![image-20191124164139507](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164139507.png)

上传文件，失败！！

![image-20191124164211269](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164211269.png)

删除失败！！

![image-20191124164239669](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164239669.png)

新建文件夹失败！！

![image-20191124164339202](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164339202.png)

## 第二种方式：第三方工具

工具比较多，此处用winscp免费好用

选择FTP，输入账号密码，其他选项选填

![image-20191124164710498](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164710498.png)

右侧为服务器上面文件，左侧为本机文件，你可以进行如第一种方法一样的操作。

![image-20191124164801972](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124164801972.png)

## 第三种方式：Linux平台

在本机（192.168.30.80）上登录，通过FTP访问本机，其他同样的方法

![image-20191124165200532](C:\Users\Tcarry\AppData\Roaming\Typora\typora-user-images\image-20191124165200532.png)

执行以下命令进行登录：

~~~
ftp 192.168.30.80
~~~

~~~
[root@DL ~]# ftp 192.168.30.80
Connected to 192.168.30.80 (192.168.30.80).
220 (vsFTPd 3.0.2)
Name (192.168.30.80:root): dlUser
331 Please specify the password.
Password: #输入密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
~~~

登陆成功后，进行的操作

~~~
ftp>ls   #查看当前文件夹下的文件及文件夹
ftp> mkdir dir  #创建文件夹
ftp>cd /dir   #进入dir文件夹
ftp>get /dir/test.txt /home/tt.txt   #将dir文件夹下的test.txt下载到本机的home下,名称改为tt.txt
ftp>put /home/tt.txt /dir/up.txt   #将home文件夹下的tt.txt上传到服务器的dir下,名称改为up.txt
ftp>mget #批量下载，用通配符*表示
ftp>mput #批量上传，用通配符*表示
ftp>bye  #退出服务器，exit和quit都可以
~~~

查看

~~~
ftp> ls
227 Entering Passive Mode (192,168,30,80,231,78).
150 Here comes the directory listing.
drwxr-xr-x    2 1006     1002           34 Nov 24 07:49 dl
~~~

新建文件夹

~~~
ftp> mkdir dir
257 "/dir" created
~~~

上传文件

~~~
ftp> put /home/DL/test.txt /dir/tt.txt
local: /home/DL/test.txt remote: /dir/tt.txt
227 Entering Passive Mode (192,168,30,80,215,84).
150 Ok to send data.
226 Transfer complete.
152 bytes sent in 5.8e-05 secs (2620.69 Kbytes/sec)
~~~

下载文件

~~~
ftp> get /dir/tt.txt /home/DL/up.txt
local: /home/DL/up.txt remote: /dir/tt.txt
227 Entering Passive Mode (192,168,30,80,134,111).
150 Opening BINARY mode data connection for /dir/tt.txt (152 bytes).
226 Transfer complete.
152 bytes received in 4.9e-05 secs (3102.04 Kbytes/sec)
~~~

删除文件

~~~
ftp> delete tt.txt
250 Delete operation successful.
~~~

删除文件夹

~~~
ftp> rmdir dir
250 Remove directory operation successful.
~~~



------------

### Are you get it ?






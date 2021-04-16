# git在IDEA的安装使用 （windows 10）

1. 官网下载git并安装，安装后将bin目录配制到环境变量path中
2. GitHub创建仓库

![image-20191209162204697](.\img\image-20191209162204697.png)

3.添加public key

![image-20191209162343033](.\img\image-20191209162343033.png)

4. 获得key

   - 打开git bash

   - ```shell
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```

     ```shell
     Generating public/private rsa key pair.
     ```

   - 按照提示，enter，输入两次密码

   - ```shell
     Enter a file in which to save the key (/c/Users/you/.ssh/id_rsa):[Press enter]
     ```

   - ```shell
     Enter passphrase (empty for no passphrase): [Type a passphrase]
     > Enter same passphrase again: [Type passphrase again]
     ```

   - 打开c/Users/you/.ssh/下的id_rsa.pub，打开复制

5. 添加key

   ![image-20191209162850729](.\img\image-20191209162850729.png)

![image-20191209162939928](.\img\image-20191209162939928.png)

-----

IDEA实操

~~~
E:\Idea_workspace\datamanager>git init
Initialized empty Git repository in E:/Idea_workspace/datamanager/.git/

E:\Idea_workspace\datamanager>git status
On branch master
No commits yet


Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        .mvn/
        mvnw
        mvnw.cmd
        pom.xml
        src/

nothing added to commit but untracked files present (use "git add" to track)

E:\Idea_workspace\datamanager>git add .
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in .mvn/wrapper/MavenWrapperDownloader.java.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in .mvn/wrapper/maven-wrapper.properties.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in mvnw.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in mvnw.cmd.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in pom.xml.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in src/main/java/com/whu/datamanager/DatamanagerApplication.java.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in src/main/resources/application.properties.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in src/test/java/com/whu/datamanager/DatamanagerApplicationTests.java.
The file will have its original line endings in your working directory

E:\Idea_workspace\datamanager>git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitignore
        new file:   .mvn/wrapper/MavenWrapperDownloader.java
        new file:   .mvn/wrapper/maven-wrapper.jar
        new file:   .mvn/wrapper/maven-wrapper.properties
        new file:   mvnw
        new file:   mvnw.cmd
        new file:   pom.xml
        new file:   src/main/java/com/whu/datamanager/DatamanagerApplication.java
        new file:   src/main/java/com/whu/datamanager/controller/loginController.java
        new file:   src/main/resources/application.properties
        new file:   src/test/java/com/whu/datamanager/DatamanagerApplicationTests.java


E:\Idea_workspace\datamanager>git commit -m "first Hello!"

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'Tcarry@DESKTOP-CCF0L8N.(none)')

E:\Idea_workspace\datamanager>git config --global user.email "1084484001@qq.com"

E:\Idea_workspace\datamanager>git commit -m "first Hello!"
[master (root-commit) 51519d8] first Hello!
 11 files changed, 768 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .mvn/wrapper/MavenWrapperDownloader.java
 create mode 100644 .mvn/wrapper/maven-wrapper.jar
 create mode 100644 .mvn/wrapper/maven-wrapper.properties
 create mode 100644 mvnw
 create mode 100644 mvnw.cmd
 create mode 100644 pom.xml
 create mode 100644 src/main/java/com/whu/datamanager/DatamanagerApplication.java
 create mode 100644 src/main/java/com/whu/datamanager/controller/loginController.java
 create mode 100644 src/main/resources/application.properties
 create mode 100644 src/test/java/com/whu/datamanager/DatamanagerApplicationTests.java
E:\Idea_workspace\datamanager>git remote add origin git@github.com:OrangeSpatial/learn.git

E:\Idea_workspace\datamanager>git push -u origin master
The authenticity of host 'github.com (111.48.9.213)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,111.48.9.213' (RSA) to the list of known hosts.
Enter passphrase for key '/c/Users/cheng/.ssh/id_rsa':
ERROR: The key you are authenticating with has been marked as read only.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

#出错是因为添加key的时候下面没有打钩，口 Allow write access

E:\Idea_workspace\datamanager>git push -u origin master
Enter passphrase for key '/c/Users/cheng/.ssh/id_rsa':
Enumerating objects: 28, done.
Counting objects: 100% (28/28), done.
Delta compression using up to 4 threads
Compressing objects: 100% (18/18), done.
Writing objects: 100% (28/28), 52.86 KiB | 94.00 KiB/s, done.
Total 28 (delta 0), reused 0 (delta 0)
To github.com:OrangeSpatial/learn.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.

完成提交！！
~~~



一些命令：

第一次进行了 

~~~
git commit -m "信息"
~~~

修改一点后再次提交

~~~
git add .
gitcommit --amend --no-edit
git push
~~~


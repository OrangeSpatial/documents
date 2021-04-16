# Ubuntu16.04 深度学习环境

https://blog.csdn.net/qq_40806289/article/details/82683034



\1. ubuntu 16.04默认安装了第三方开源的驱动程序nouveau，安装nvidia显卡驱动首先需要禁用nouveau，不然会碰到冲突的问题，导致无法安装nvidia显卡驱动。

编辑文件blacklist.conf

sudo vim /etc/modprobe.d/blacklist.conf
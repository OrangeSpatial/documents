# Ubuntu安装rtx 2080ti 显卡

vsphere 6.7 

## 查看显卡是否存在

~~~
# 更新内核版本
sudo update-pciids
# 查看pci
lspci | grep -i nvidia
~~~

## 安装依赖

~~~cmd
sudo apt-get install gcc g++ make
~~~

## 停止使用桌面环境

通过`Ctrl+Alt+F3`（F1-F6）快捷键打开终端

~~~cmd
sudo telinit 3
~~~

## 安装

~~~cmd
sudo chmod +x NVIDIA-Linux-x86_64-430.26.run # 添加执行权限
sudo bash NVIDIA-Linux-x86_64-430.26.run –no-opengl-files –no-x-check
~~~

提示中：

继续

忽视 gcc

ok

warning... --ok

no

ok

## 验证

~~~
nvidia-smi
~~~

不出意外会出现失败！

配制虚拟机选项参数

~~~
hypervisor.cpuid.v0 = FALSE
~~~

启动
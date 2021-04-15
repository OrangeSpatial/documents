# Ubuntu 18.04.4 Jupyter实现远程访问，nginx代理访问，开机启动

##  一、安装

jupyter安装直接使用Anaconda安装即可，见Anaconda安装。

~~~cmd
$ conda 创建虚拟环境

$ conda env list 或 conda info -e 查看当前存在哪些虚拟环境
# 创建
$ conda create -n myenv python=3.7.6
# 激活虚拟环境
$ source activate myenv
# 退出
$ conda deactivate
# 删除
$ conda remove --name myenv  package_name
~~~



## 二、配制jupyter

执行下面命令生成配制文件

~~~cmd
jupyter notebook --generate-config

# 修改配制文件内容
# 该配置文件位置如果直接放在用户目录下和.jupyter(默认生成位置)，则启动时优先读取用户目录下的
vim ~/.jupyter/jupyter_notebook_config.py

# 修改一下内容
#允许远程访问
c.NotebookApp.allow_remote_access = True

#修改访问根路径
c.NotebookApp.base_url = '/jupyter60'

# 监听的Ip地址
c.NotebookApp.ip = '*'

#不自动打开浏览器
c.NotebookApp.open_browser = False

#配制生成的密码
c.NotebookApp.password = 'sha1:64aba78fdb3c:f4ef22f4a3262ef5621ae0743189dc80ef5bba1d'

# 去掉密码验证
c.NotebookApp.token = ""

# 端口
c.NotebookApp.port = 8888

# 是否开启新建终端
c.NotebookApp.terminals_enabled = False

# 是否可以通过前端修改密码
c.NotebookApp.allow_password_change = False

# 前端是否展示退出按钮
c.NotebookApp.quit_button = False

# 默认打开的目录路径
c.NotebookApp.notebook_dir = "~/dl"

# 关闭自动退出功能
c.NotebookApp.shutdown_no_activity_timeout = 0 

# 内嵌跨域

c.NotebookApp.tornado_settings = {
	'headers': {
            'Content-Security-Policy': "frame-ancestors self *; report-uri /api/security/csp-report",
      }
}

~~~

启动写本机ip，默认localhost或27.0.0.1，局域网也无法访问

~~~cmd
jupyter notebook --ip 192.168.10.102 
~~~

## 三、nginx 代理

以下是配制文件 本次是使用docker部署的nginx服务

修改conf.d下的default.conf即可，配置中61基本就可以，不行可以试试60

~~~json
# 负载均衡（本次未使用）
upstream group3 {
    server 192.168.30.93:8888;
    server 192.168.30.94:8888;
}


server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html/test;
        index  index.html index.htm;
    }


    location /jupyter61 {
	proxy_pass	http://192.168.30.61:8888;
	#proxy_redirect	default;
	proxy_set_header Host $host;
        proxy_set_header X-FORWARDED-FOR $remote_addr;
    }

   location ~ /jupyter61/api/kernels/ {
      proxy_pass            http://192.168.30.61:8888;
      proxy_set_header      Host $host;
      # websocket support
      proxy_http_version    1.1;
      proxy_set_header      Upgrade "websocket";
      proxy_set_header      Connection "Upgrade";
      proxy_read_timeout    86400;
   }

   location ~ /jupyter61/terminals/ {
      proxy_pass            http://192.168.30.61:8888;
      proxy_set_header      Host $host;
      # websocket support
      proxy_http_version    1.1;
      proxy_set_header      Upgrade "websocket";
      proxy_set_header      Connection "Upgrade";
      proxy_read_timeout    86400;
   }

   location ~ /jupyter61/api/contents/ {
      proxy_pass            http://192.168.30.61:8888;
      proxy_set_header      Host $host;
      # websocket support
      proxy_http_version    1.1;
      proxy_set_header      Upgrade "websocket";
      proxy_set_header      Connection "Upgrade";
      proxy_read_timeout    86400;
   }

	# /jupyter60 是jupyter中配制的访问根路径
    location /jupyter60 {
        proxy_pass      http://192.168.30.60:8888;
        proxy_set_header Host $host;
        proxy_set_header X-FORWARDED-FOR $remote_addr;
    }


    location ~ /jupyter60/api/kernels/ {
       proxy_pass            http://192.168.30.60:8888;
       proxy_set_header      Host $host;
        # websocket support
       proxy_http_version    1.1;
       proxy_set_header      Upgrade "websocket";
       proxy_set_header      Connection "Upgrade";
       proxy_read_timeout    86400;
    }





    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

~~~

接下来可以直接用nginx部署IP/jupyter60 => http://IP/jupyter60

## 四、服务自启动

~~~cmd
# 创建并编辑 
sudo vim /lib/systemd/system/jupyterd.service
~~~

~~~cmd
[Unit]
Description=jupyter notebook
After=network.target
[Service]
Type=simple
# cheng 为普通用户名
User=cheng
# jupyter-notebook 位置 可以使用（whereis jupyter-notebook）命令查看
EnvironmentFile=/home/cheng/anaconda3/bin/jupyter-notebook
# 启动命令，如果配置了默认路径，后面的/home/cheng/dl不用配制
ExecStart=/home/cheng/anaconda3/bin/jupyter-notebook --ip 192.168.10.102 /home/cheng/dl
# 关闭服务命令
ExecStop=/usr/bin/pkill /home/cheng/anaconda3/bin/jupyter-notebook
KillMode=process
Restart=on-failure
RestartSec=30s
[Install]
WantedBy=multi-user.target
~~~

~~~cmd
[Unit]
Description=jupyter notebook
After=network.target
[Service]
Type=simple
# 这里填用户名，下同
User=cheng
Group=cheng
# EnvironmentFile=/home/cheng/anaconda3/bin/jupyter-notebook
WorkingDirectory=/home/cheng/dl
#ExecStartPre=. activate myenv
ExecStart=/home/cheng/anaconda3/bin/jupyter-notebook --ip 192.168.10.102 /home/cheng/dl --config=/home/cheng/.jupyter/jupyter_notebook_config.py
#ExecStop=/usr/bin/pkill /home/cheng/anaconda3/bin/jupyter-notebook
#KillMode=process
Restart=on-failure
RestartSec=10s
[Install]
WantedBy=multi-user.target
~~~



服务开机启动及其他操作

~~~cmd
# 加载
sudo systemctl daemon-reload
# 开机启动
sudo systemctl enable jupyterd
# 启动
sudo systemctl start jupyterd
# 停止
sudo systemctl stop jupyterd
# 查看状态
sudo systemctl status jupyterd
~~~

如果服务有问题，移除 重新配制

~~~cmd
# 停止服务
sudo systemctl stop jupyterd

# 删除/lib/systemd/system/jupyter.service
sudo rm /lib/systemd/system/jupyterd.service

# 重新加载即可
sudo systemctl daemon-reload

# 再执行启动，将会发现找不到
sudo systemctl start jupyterd
~~~

## end！
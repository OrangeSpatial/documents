# springboot 和Vue  Nginx负载均衡

部署方式 docker

部署四个springboot服务 端口分别为8081,8082,8083,8084

nginx部署

负载均衡到同一端口 8003

部署vue到8001



## 部署springboot

创建Dockerfile

~~~
# Docker image for springboot file run
# VERSION 0.0.1
# Author: eangulee
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER orange <xx@xx.com>
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp 
# 将jar包添加到容器中并更名为app.jar
ADD welearn-0.0.1-SNAPSHOT.jar app.jar 
# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
~~~

在服务器新建一个docker文件夹，将maven打包好的jar包和Dockerfile文件复制到服务器的docker文件夹下

创建镜像

~~~
docker build -t springboot4welearn .
~~~

~~~
-d 参数是让容器后台运行 
-p 是做端口映射，此时将服务器中的8080端口映射到容器中的8085(项目中端口配置的是8085)端口
docker run -d --name=welearn1 -p 8081:8003 springboot4welearn
docker run -d --name=welearn2 -p 8082:8003 springboot4welearn
docker run -d --name=welearn3 -p 8083:8003 springboot4welearn
docker run -d --name=welearn4 -p 8084:8003 springboot4welearn
~~~

~~~
#查看容器
docker ps

#查看日志
docker logs -f id
~~~



## nginx反向代理+负载均衡

docker拉去最新nginx镜像

~~~sh
docker search nginx
docker pull nginx:latest
docker images

docker run --name nginx4welearn -p 8002:8888 -d nginx
docker run --name nginx4minio --restart=always -p 8002:9000 -d nginx

~~~

创建挂载目录

~~~sh
mkdir -p /data/nginx_welearn/{conf,conf.d,html,logs}
~~~

~~~sh
docker cp 13d3a80e893f:/etc/nginx/nginx.conf /data/nginx_welearn/conf
docker cp 13d3a80e893f:/usr/share/nginx/html/index.html /data/nginx_welearn/html
docker cp 13d3a80e893f:/etc/nginx/conf.d/default.conf /data/nginx_welearn/conf.d
~~~



运行容器并挂载目录

~~~sh
docker run --name nginx_welearn -d -p 8003:80 -p 8001:8080 --restart=always  -v /data/nginx_welearn/conf/nginx.conf:/etc/nginx/nginx.conf  -v /data/nginx_welearn/logs:/var/log/nginx -v /data/nginx_welearn/html:/usr/share/nginx/html -v /data/nginx_welearn/conf.d/default.conf:/etc/nginx/conf.d/default.conf nginx
~~~

修改/data/nginx_welearn/conf.d/default.conf

~~~conf

    upstream welearn_server {
        server 192.168.30.111:8081;
        server 192.168.30.111:8082;
        server 192.168.30.111:8083;
        server 192.168.30.111:8084;
    }

    server {
        listen        80;
        server_name  localhost;

        location /index {
                root /usr/share/nginx/html/dist;
                index index.html;
        }

    }
    
    server {
        listen        80;
        server_name  localhost;
        
        proxy_set_header Cookie $http_cookie;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
        
        location / {
                root /usr/share/nginx/html/dist;
                index index.html;
        }

        location / {
            # 前端请求: /api/login 代理后: http://127.0.0.1:8080/login
            proxy_pass http://welearn_server/;
            # 解决springboot中获取远程ip的问题
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
    server {
        listen        8080;
        server_name  localhost;
        
        proxy_set_header Cookie $http_cookie;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

        location / {
            # 前端请求: /api/login 代理后: http://127.0.0.1:8080/login
            proxy_pass http://welearn_server/;
            # 解决springboot中获取远程ip的问题
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }

    }
~~~






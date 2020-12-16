---
layout: post
title: IDEA 部署Docker镜像
category: docker
tags: [docker]
---



### Docker 

开启虚拟机，打开 finalshell 客户端

- 开启docker `service docker start`

#### 镜像操作

- 查看镜像 `docker images`
- 搜索镜像 `docker serarch 镜像名`
- 拉取镜像 `docker pull 镜像名:标签号` 例如： `docker pull mysql:5.7.26`
- 删除镜像 `docker rmi 镜像ID/名称`

#### 容器操作

- 查询容器 `docker ps `, 查看所有容器`docker ps -a`
- 删除容器 `docker rm 容器ID/容器名`

- 开启容器 `docker start 容器ID/容器名`
- 停止容器 `docker stop 容器ID/容器名`
- 进入容器操作 `docker exec -it 容器ID bash`
- 查看容器日志 `docker logs Name/ID`
- 拷贝文件到宿主机 ` docker cp -a bb397b55cde0:/tmp/ /tmp`
- 拷贝文件到容器 ` docker cp -a /tmp bb397b55cde0:/tmp/`



#### 开启Docker的远程连接

1. 编辑文件 docker.service

   `vi /usr/lib/systemd/system/docker.service`

   找到 `ExecStart=/usr/bin/dockerd`这一行

   将其改为如下内容：

   `ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock`

2. 重新加载配置文件

   `systemctl daemon-reload`

3. 重启docker

   `systemctl restart docker`

4. 查看 2375 端口是否开放

   `netstat -nlpt`

   也可以直接访问 `curl http://127.0.0.1:2375/info`，如果有返回信息，则已开放

   

#### IDEA 下载 插件

PS：*总觉得这个插件有bug，如在 容器里设置了参数不生效。。。*

在 Settings->Plugins->Marketplace 搜索 docker

![image-20201216155142769](https://gitee.com/szwei/images/raw/master/img/image-20201216155142769.png)

安装完重启 IDEA



然后在 Settings 里搜索 docker ，配置上docker 地址，下面显示 successful 即连接成功 

![image-20201216155406515](https://gitee.com/szwei/images/raw/master/img/image-20201216155406515.png)



在 IDEA 页面的下面 有个 docker 的按钮，点开可以看到docker里所有的容器，所有的镜像

![image-20201216155742277](https://gitee.com/szwei/images/raw/master/img/image-20201216155742277.png)



#### IDEA 推送镜像

首先maven clean 清理本地代码

maven package 打包本地项目，生成 jar 

在 DockerFile 页面，编辑

![image-20201216160324772](https://gitee.com/szwei/images/raw/master/img/image-20201216160324772.png)

 `/usr/share/fonts/dejavu/`

如果需要 docker里的mysql ,则 在配置文件里 这样写即可：

`url: jdbc:mysql://mysql:3306/DBname?characterEncoding=utf8&useSSL=false&allowMultiQueries=true&serverTimezone=Asia/Shanghai`

点击运行就可以将上面生成的 jar 打包成 docker 镜像，并上传到 docker 里。

如果项目依赖较多，需要较长时间

在IDEA 控制台就可以看到上传成功的 镜像，可以新建容器，运行

在IDEA 控制台也可以配置 容器的参数，比如端口映射，挂载文件，启动日志等，十分方便。



就酱，后台项目就启动完成了，物理机访问虚拟机ip 加上映射出来的端口号就可以访问到接口了

不过，仅能访问后台接口不是我们最终要的效果，得有界面啊！



#### 安装MySQL

1. 拉取镜像`docker pull mysql:5.7.26`

2. 启动`docker run -p 3306:3306 --name mysql:5.7.26 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.26`

3. 配置外网访问

   进入容器 ` docker exec -it 47e74a93cf87 bash`

   登录`mysql -uroot -p`输入密码

   命令 ` grant all privileges on *.* to root@"%" identified by "123456" with grant option;`

   刷新 `flush privileges; `

   之后就可以我们就可以在物理机上通过nvicate 来连接上。

   

#### 安装nginx

1. 拉取镜像: `docker pull nginx`

2. 查看镜像: `docker images`

3. 在宿主机创建配置文件目录

   `mkdir -p /data/nginx/{conf,conf.d,html,log}`

4. nginx.conf 

   ```bash
   #user  nobody;
   worker_processes  1;
   
   #error_log  logs/error.log;
   #error_log  logs/error.log  notice;
   #error_log  logs/error.log  info;
   
   #pid        logs/nginx.pid;
   
   
   events {
       worker_connections  1024;
   }
   
   
   http {
       include       mime.types;
       default_type  application/octet-stream;
   
       sendfile        on;
       #tcp_nopush     on;
   
       #keepalive_timeout  0;
       keepalive_timeout  65;
   
       #gzip  on;
   
       server {
           listen       80;
           server_name  localhost;
   
           #charset koi8-r;
   
           #access_log  logs/host.access.log  main;
   
           location / {
               index  index.html index.htm;
               root /usr/share/nginx/html;
                charset utf-8;
                 try_files $uri $uri/ /index.html;
           }
           	location ^~/api/ {
                 proxy_pass http://192.168.1.130:8080/;
           }
   
           #error_page  404              /404.html;
   
           # redirect server error pages to the static page /50x.html
           #
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   html;
           }
       }
   
   }
   
   ```

   

5. index.html

   ```html
   <!DOCTYPE html>
   <html>
   <head>
   <meta charset="utf-8">
   <title>docker搭建nginx</title>
   </head>
   <body>
       <h1>docker搭建nginx映射成功</h1>
   </body>
   </html>
   ```

   

6. 启动并挂载配置文件目录

   `docker run --name my_nginx -d -p 80:80  -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /data/nginx/log:/var/log/nginx -v /data/nginx/html:/usr/share/nginx/html nginx`

7. 在物理机访问 虚拟机ip ,看是否成功

8. 重启nginx`docker restart 容器ID`

   

#### 部署项目

1. 将前端项目，build ，将生成的 dist 文件夹里的内容上传到 html 目录
2. 在 nginx.conf 配置文件里配置后台接口地址
3. 访问nginx 服务器ip，测试是否可以访问。



#### 远程访问

1. 经过上面的操作，我们已经可以在本地跑起来idea 里的项目了，在idea控制台也可以直接看到项目的运行日志。

2. 那么，如果我们要给领导或者客户看项目的进度怎么办？

3. 可以使用 内网穿透，将局域网的 ip 映射出去。

4. 之前使用的 ngrok.cc， 免费的总是断，体验不是很好。推荐一个稳定点的。utools，里面有内网穿透的功能，目前来说还是挺快挺好用的。

5. 使用界面如下图所示：

6. 完结撒花~~

   ![image-20201216180509153](https://gitee.com/szwei/images/raw/master/img/image-20201216180509153.png) 



### Docker 容器中文字体乱码问题

​	在宿主机内安装中文字体

1. 查看已经安装的中文字体 ` fc-list :lang=zh `

2. 如果没有这个命令，安装上 ` yum -y install fontconfig`

3. 将Windows 上的 C:\Windows\Fonts 文件夹下的字体文件上传到 服务器目录  `/usr/share/fonts/dejavu/`下

   ​	比如 ： 宋体字 (simsun.ttc)

4. 清除缓存 `fc-cache`

然后在需要中文字体的容器里挂载宿主机的 字体文件, ==，冒号前为宿主主机目录，冒号后为容器对应目录==

` docker run -p 80:80--name demo -v /usr/share/fonts/dejavu/:/usr/share/fonts/`


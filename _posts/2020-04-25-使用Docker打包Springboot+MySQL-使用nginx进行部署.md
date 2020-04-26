---
layout: post
title: "使用Docker+maven对Springboot+MySQL项目进行打包，结合nginx部署"
subtitle: ""
date: 2020-04-25
author: Aether
category: coding
tags: CODING
finished: true
---

#使用Docker+maven对Springboot+MySQL项目进行打包与部署

> 在SE的软件架构与中间件课程的lab2中，需要将springboot项目部署到虚拟机+真实服务器上。正好借由这个机会学习一下docker的使用，同时也正好使部署变得简单快捷一些。

# nginx 

## 配置 nginx

~~~shell
sudo apt-get update
sudo apt-get install nginx
~~~

## 修改nginx配置文件`/etc/nginx/nginx.conf`

负载均衡器`192.168.0.110`配置 ：

~~~
upstream nodes {
	server 192.168.0.111:8080;
	server 192.168.0.112:8080;
}
server {
	listen		8080;
	server_name 192.168.0.110; # 在实际配置中替换为公网IP
	location / {
		proxy_pass http://nodes;
		include proxy_params;
	}
}
~~~

服务器`192.168.0.111`配置：

~~~
server {
	listen		8080;
  server_name 192.168.0.111;
}

~~~

## 配置nginx代理参数文件`/etc/nginx/proxy_params`

~~~
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 
proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;
 
proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;
~~~

## nginx启动、重启等操作

~~~
sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx stop
sudo /etc/init.d/nginx restart
sudo /etc/init.d/nginx reload
~~~

此时对`192.168.0.110:8080`的访问将会分配到`192.168.0.111:8080`和`192.168.0.112:8080`



# Docker

## 建立容器间的网络连接

~~~
docker network create  external-api
~~~

若忽略此步骤，则打包后的`mysql` 和`springboot`的`docker`镜像无法连接

参考 https://stackoverflow.com/questions/54692666/docker-java-net-connectexception-connection-refused-application-running-at/54692798

## 使用docker拉取mysql镜像

~~~
Docker pull mysql:8
~~~

## 运行mysql

~~~
docker run --net external-api --name ly-mysql --restart unless-stopped -e MYSQL_ROOT_PASSWORD=123456789 -d mysql:8
~~~

## 导入数据库

~~~
docker exec -i ly-mysql mysql -u root --password=123456789 < lab1/sale.sql  
~~~

---

## 使用maven对springboot项目打包

在IDEA中，使用`maven`将项目打包为`war`

> Maven --> Lifecycle --> package 

此过程中遇到的错误:项目打成`war`包后缺失`webapp`目录下的资源

参考https://blog.csdn.net/sanri1993/article/details/52201372

## 配置Dockerfile

~~~dockerfile
FROM openjdk:8
ADD HitSoftSaleManage-0.0.1-SNAPSHOT.war app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
~~~

## 使用docker构建镜像

~~~shell
docker build -t ly . && docker run -p 8080:8080 --name lab2project --net external-api ly
~~~

## 提交打包好的镜像

 ~~~
docker commit -a "lvyue" -m "project" 10532f64dc6c ly-sprintboot:v1
docker push $(mydockerusername)/ly-springboot:v1
 ~~~

---

> 以下内容为部署

## 登陆docker账号

~~~
sudo docker login -u $(mydockerusername) -p $(password)
~~~

## 拉取镜像

~~~
sudo docker image pull $(mydockerusername)/ly-mysql:v1
sudo docker image pull $(mydockerusername)/ly-springboot:v1
~~~

## 建立容器间的网络连接

~~~
docker network create  external-api
~~~

## 运行mysql容器

~~~
sudo docker run --net external-api --name ly-mysql --restart unless-stopped -e MYSQL_ROOT_PASSWORD=123456789 -d $(mydockerusername)/ly-mysql:v1
~~~

## 导入数据库

~~~
sudo docker exec -i ly-mysql mysql -u root --password=123456789 < sale.sql
~~~

## 运行springboot容器

~~~
sudo docker run -p 8080:8080 --name lab2project --net external-api $(mydockerusername)/ly-springboot:v1
~~~

---

# 附加：使用ab进行POST压力测试

~~~
ab -T "application/json" -p data -n 500 -k -c 100 192.168.0.110:8080/sale/login
~~~

`data`文件中保存POST的json数据

~~~
{
	"staff_name":"hang",
	"pass":"123123"
}

~~~


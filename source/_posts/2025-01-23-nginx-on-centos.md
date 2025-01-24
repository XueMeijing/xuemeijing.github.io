---
title: 在CentOS上安装Nginx
date: 2025-01-23 11:19:03
categories: 
- 有趣
---

一直以来，我都是使用Debian系统，用于搭建个人网站或者其他项目，中途也接触过CentOS，感觉使用上还是有些区别的，但是没有更深入的了解，正好昨天需要在客户CentOS上安装Nginx，借此机会记录一下

## 安装Nginx

与Debian使用apt包管理工具不同，CentOS使用yum。但是在CentOS上安装Nginx之前，需要先安装epel-release。

EPEL（Extra Packages for Enterprise Linux）是由 Fedora 社区打造的额外软件包仓库，提供了许多基本仓库中没有的软件包。CentOS的基础软件仓库中并不包含Nginx，所以我们需要先启用EPEL仓库：

```bash
sudo yum install epel-release
sudo yum install nginx
```
如果遇到权限问题，请确保使用sudo执行上述命令。

## 启动Nginx

之前我都是使用service nginx start 启动Nginx，但是CentOS上没有这个命令，所以需要使用systemctl
```bash
sudo systemctl start nginx
sudo systemctl status nginx
```

## 测试Nginx

按照过往经验，nginx启动之后就能正常运行了，访问ip就能看到nginx的欢迎页面，但是现在访问ip显示refused to connect

![Image](https://github.com/user-attachments/assets/69827f54-3eb7-4e35-ab6f-17fb83e30552)

但是Nginx是正常启动的啊，curl 127.0.0.1 也是正常返回的

![Image](https://github.com/user-attachments/assets/7b36a83c-d929-4db7-a7c6-3c334caf3e63)

应该是防火墙的问题了，之前在腾讯云、谷歌云的服务器，默认都是开放80端口的

## 开放防火墙

CentOS使用firewalld管理防火墙，所以需要开放防火墙
```bash
  firewall-cmd --permanent --add-service=http
  firewall-cmd --permanent --add-port=80/tcp
  firewall-cmd --reload
```
OK显示正常了，但是竟然是CentOS欢迎页，而不是nginx的欢迎页
![Image](https://github.com/user-attachments/assets/e134d41d-b887-45c1-9131-7dd661fc8ce1)

## 配置Nginx

在Debian的/etc/nginx/nginx.conf中，有以下两行，说明nginx的配置有两个来源有两个文件夹，正常需要将配置文件放在/etc/nginx/sites-available/，然后使用ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/ 创建软链接，然后重启Nginx，当然你像CentOS一样直接放在/etc/nginx/conf.d/也是可以的，只是不建议这么做

```bash
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

在CentOS的/etc/nginx/nginx.conf中，有以下一行，所以只需要将配置文件放在/etc/nginx/conf.d/，然后重启nginx就可以生效了

```bash
include /etc/nginx/conf.d/*.conf;
```

## 主要目录

他们的主要目录如下
```bash
# Debian/Ubuntu
主配置文件：/etc/nginx/nginx.conf
站点配置：/etc/nginx/sites-available/
启用的站点：/etc/nginx/sites-enabled/
默认站点目录：/var/www/html/

# CentOS
主配置文件：/etc/nginx/nginx.conf
站点配置：/etc/nginx/conf.d/
默认站点目录：/usr/share/nginx/html/
```

为什么CentOS版本的Nginx的欢迎页面是CentOS的欢迎页面，而不是Nginx的欢迎页面呢？看一下CentOS的默认目录
![Image](https://github.com/user-attachments/assets/282f3a1c-1f1a-46e2-88a6-84042f05ebe8)

/usr/share/nginx/html/ 下的index.html是一个软连接，指向了 /usr/share/doc/HTML/index.html ，也就是CentOS的欢迎页面

## 使用

使用就简单了，在/etc/nginx/conf.d/ 下创建一个文件test.conf，然后写入以下配置

```
server {
        listen 5173 default_server;
        listen [::]:5173 default_server;

        #
        root /usr/share/nginx/test;
        #
        index index.html index.htm;

        server_name _;

        location / {
                try_files $uri $uri/ /index.html;
        }
}
```

如果端口5173没有开放，还要在防火墙放开一下这个端口

```bash
  firewall-cmd --permanent --add-port=5173/tcp
  firewall-cmd --reload
```

把前端打包后的文件放到/usr/share/nginx/test下，然后重启nginx就可以生效了，这时候访问ip:5173就可以看到前端页面了

重启Nginx的命令：
```bash
sudo systemctl restart nginx
```

检查配置文件语法是否正确：
```bash
sudo nginx -t
```

如果需要查看Nginx的错误日志：
```bash
sudo tail -f /var/log/nginx/error.log
```
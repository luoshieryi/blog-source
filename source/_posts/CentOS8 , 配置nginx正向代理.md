---
title: Linux下使用Nginx配置正向代理
date: 2022-02-27
tags: [linux, nginx, centos, proxy]
---

# Linux下使用Nginx配置正向代理123

## 前言

Nginx 除了用于搭建反向代理, 负载均衡, 也可以用于搭建正向代理服务器

正向代理的作用有: 

- 内网服务器访问外网

- 提供 VPN 服务

## 物料准备

本文使用环境为 CentOS8 , nginx 原生不直接支持https代理, `使用 ngx_http_proxy_connect_module` 实现 https 代理

- nginx (此处使用1.19.2) : http://nginx.org/download/nginx-1.19.2.tar.gz
- nginx 1.19.2 对应 https 模块 : https://github.com/chobits/ngx_http_proxy_connect_module/archive/refs/tags/v0.0.2.zip
- gcc cpp 编译器
- pcre-devel 与 openssl-devel

## 开始安装

1. 切换到 nginx 安装目录 `cd /opt/nginx`

2. 下载模块 `wget https://github.com/chobits/ngx_http_proxy_connect_module/archive/refs/tags/v0.0.2.zip`

3. 解压 `unzip v0.0.2.zip` 

4. 下载 nginx `wget http://nginx.org/download/nginx-1.19.2.tar.gz`

5. 打入https模块

   ```bash
   tar xf nginx-1.19.2.tar.gz
   
   cd nginx-1.19.2
   
   patch -p1 < /opt/nginx/ngx_http_proxy_connect_module-0.0.2/patch/proxy_connect_rewrite_1018.patch
   ```

6. 安装 pcre-devel 与 openssl-devel `yum -y install pcre-devel openssl openssl-devel`

7. 安装gcc编译器, 编译安装 nginx 

   ```bash
   yum -y install gcc cmake make cmake unzip ncurses-devel gcc gcc-c++
   
   ./configure --prefix=/opt/nginx/nginx_app --add-module=/opt/nginx/ngx_http_proxy_connect_module-0.0.2
   
   make && make install
   ```

8. 配置 nginx

   ```bash
   cd /opt/nginx/nginx_app/conf/
   
   vim nginx.conf
   ```

   ```text
    server {
           listen                           1270;			  		#自定义正向代理使用端口
           server_name                      luoluo;				#自定义正向代理服务名称
           # dns resolver used by forward proxying
        	# 域名解析服务器
           resolver                         114.114.114.114;
           # forward proxy for CONNECT request
           proxy_connect;
           # 允许端口，all所有, 此处 80 表示 http 请求所需端口, 443 表示 https
           proxy_connect_allow              443 80;
           proxy_connect_connect_timeout    10s;
           proxy_connect_read_timeout       10s;
           proxy_connect_send_timeout       10s;
           location / {
               proxy_pass $scheme://$http_host$request_uri;
           }
       }
   ```

9. 编写 `systemd`  启动服务

   ```bash
   cat > /etc/systemd/system/nginx.service << EOF
   [Unit]
   Description=The NGINX HTTP and reverse proxy server
   After=syslog.target network-online.target remote-fs.target nss-lookup.target
   Wants=network-online.target
   
   [Service]
   Type=forking
   PIDFile=/opt/nginx/nginx_app/logs/nginx.pid
   ExecStartPre=/opt/nginx/nginx_app/sbin/nginx -t
   ExecStart=/opt/nginx/nginx_app/sbin/nginx
   ExecReload=/opt/nginx/nginx_app/sbin/nginx -s reload
   ExecStop=/bin/kill -s QUIT $MAINPID
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

10. 启动 nginx 服务

    ```bach
    systemctl daemon-reload
    systemctl start nginx
    ```

11. 开放防火墙对应端口

## 拓展

### 限定 IP 访问

在 `nginx.conf` 的 `server{}` 中添加配置

1. 仅允许 123.123.123.123 访问

    ```text
    allow 123.123.123.123
    deny all
    ```

2. 仅禁止 123.123.123.123 访问

   ```text
   deny 123.123.123.123
   ```

3. 仅允许 123.123.123.1-254 网段访问

   ```text
   allow 123.123.123.1/24
   deny all
   ```

4. 仅禁止 123.123.123.1-254 网段访问

   ```text
   deny 123.123.123.1/24
   ```

5. 重新加载配置文件 `nginx -s reload` (或重启服务)

### 限定 用户名/密码 访问

1. 安装 apacha-htpasswd
   `yum install httpd-tools -y`

2. 生成用户名密码 `htpasswd -bc 文件路径 用户名 密码`
   e.g. `htpasswd -bc /opt/nginx/psd root 123456`

3. 在 `nginx.conf` 的 `server{}` 中添加配置

   ```text
   auth_basic "input password";   # 这里是提示信息
   
   auth_basic_user_file /opt/nginx/psd; # 这里填写刚才生成的文件路径
   ```

4. 重新加载配置文件 `nginx -s reload` (或重启服务)

## 备注

- LNMP高版本脚本编译安装的nginx默认支持ipv6, 所以不必纠结nginx -V查看的结果是否有“--with-ipv6", 如果使用 "--with-ipv6" 参数, 会报 warning :  
  `./configure: warning: the “–with-ipv6” option is deprecated`


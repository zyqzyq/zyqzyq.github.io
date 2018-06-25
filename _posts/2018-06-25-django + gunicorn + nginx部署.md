---
author: zyqzyq
date: 2018-06-25 9:16:32+00:00
layout: post
title: django + gunicorn + nginx 部署
key: 20180625
tags:
- django
- gunicorn 
- nginx
---

# 创建python 虚拟环境

### 安装pipenv

```
sudo pip install pipenv 
```

### 新建项目

```
mkdir ~/myproject
cd ~/myproject
```

### 新建python环境

```
pipenv --python 3.6

```
> 此处指定python版本进行安装，需要提前安装python3.6 ，也可以直接运行 pipenv --three 进行安装

### 激活环境并安装所需的包

```
pipenv shell
pipenv install django gunicorn celery
```
> 至此环境配置完成。

# 新建并配置django项目

### 新建django 项目
进入python 虚拟环境（以后默认都在该环境下进行操作）

```
cd ~/myproject
pipenv shell
```
新建项目

```
django-admin.py startproject myproject
```
### 更改项目配置
编辑myproject/setting 文件

```
nano myproject/settings.py
```

在文件末尾添加以下内容并保存

```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```


### 初始化项目
初始化数据

```
python manage.py makemigrations
python manage.py migrate

```
新建管理员账号

```
python manage.py createsuperuser
```
收集静态文件

```
python manage.py collectstatic
```
静态文件会被集中放置到~/myproject/static 中。

最后使用django自带的轻量级服务进行测试

```
python manage.py runserver 0.0.0.0:8000
```
在浏览器中输入以下地址
```
http://server_domain_or_IP:8000
```
你可以看到默认的django主页面

### 测试gunicorn

输入以下命令测试gunicorn
```
gunicorn --bind 0.0.0.0:8000 myproject.wsgi:application
```
在浏览器中再次浏览网页，如果现实正常则说明gunicorn运行成功。

# 新建gunicorn服务

在文本编辑器中使用sudo权限创建并打开Gunicorn的启动文件

```
sudo nano /etc/init/gunicorn.conf
```
首先为我们的服务添加一个简单的描述，然后定义自动启动服务的系统运行级别


```
description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

```

接下来，我们要设置当服务停止时自动重启，并在指定的用户和组下运行，并将执行目录更改为我们的项目目录

```
description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
setuid user
setgid www-data
chdir /home/user/myproject
```
现在，添加启动gunicorn的命令，我们需要给出虚拟环境中gunicorn可执行文件的路径。我们选择使用了unix套接字和nginx进行通信，相比较于网络端口这更安全，更快速。

```
description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
setuid user
setgid www-data
chdir /home/user/myproject

exec myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/user/myproject/myproject.sock myproject.wsgi:application
```

保存文件，然后启动gunicorn服务

```
sudo service gunicorn start
```

# 配置nginx代理gunicorn

新建nginx配置文件

```
sudo nano /etc/nginx/sites-available/myproject
```
在文件中，新建一个新的服务器，监听80端口并对服务器的ip或者域名做出响应。

```
server {
    listen 80;
    server_name server_domain_or_IP;
}
```
接下来，我们需要设置忽略favicon相关的错误，然后设置静态文件的根目录。

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }
}
```
最后，我们会创建一个location / {}匹配所有其他请求，将流量通过套接字传递到gunicorn进程。

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/user/myproject/myproject.sock;
    }
}
```
保存文件。通过文件链接来启动该文件

```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```
测试nginx 配置

```
sudo nginx -t
```
如果没有错误，重启nginx服务

```
sudo service nginx restart
```

大功告成。

参考网址：[https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn)









[TOC]

# typecho 网站搭建

## LNMP 环境安装

[手动搭建 LNMP 环境（CentOS 8）](https://cloud.tencent.com/document/product/213/49304)

因为安装 typecho 需要依赖 LNMP 环境，所以无脑按照腾讯云的步骤来就可以了。

## typecho 安装

### 登陆 mysql 创建数据库

待会需要用到创建的数据库名

```sql
create database my_blog;
```

### 上传  typecho.zip 文件

在 linux 机器上创建好网站目录，并上传下载好的 [typecho.zip](http://typecho.org/download) 文件。不建议创建目录在 /root 下面。我试过发现怎么也访问不到，后面换其他目录就好了。

这里我们选择上传文件到 /home/www/myblog/ 下（没有就自己创建目录）。解压 typecho.zip 后再添加权限 `chmod -R 777 myblog`。因为待会安装需要网站目录的读写权限。

### 修改 default.conf 文件

```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /data/www/myblog; # 修改为你的网站目录
        index  index.php index.html index.htm; # 这里把 index.php 放在第一位，这样子 / 会默认优先访问 index.php
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
    # Nginx服务器无法登录后台，点击前台链接或者后台登录时出现"404, not found"
    # 一般的出现这种情况时,nginx.conf里的的location设置都是类似这样
    # location ~ .*\.php$
    # 要支持pathinfo，要改成
    # location ~ .*\.php(\/.*)*$
    # 在某些老版本的php里面，可能还要打开php.ini里的cgi.fix_pathinfo
	# cgi.fix_pathinfo = 1
    location ~ .*\.php(\/.*)*$ { # location ~ \.php$ 修改为 location ~ .*\.php(\/.*)*$
        root           /data/www/myblog; # 修改为你的网站目录
        fastcgi_pass   unix:/run/php-fpm/www.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

修改配置文件后记得重启一下 nginx 使配置生效 `systemctl restart nginx`

### 开始安装 typecho

在浏览器输入 `ip/install.php` 进入安装步骤。一开始我们创建的数据库名这里就派上用场了。填入完信息，大功告成。


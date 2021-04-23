# nginx

## 1. 在linux上使用nginx

### 1.1 yum安装

`yum install nginx`

### 1.2 开启服务

`systemctl  start nginx.service`

### 1.3 开机启动

`systemctl enable nginx.service`

## 2. docker

### 2.1 进入Nignx

`docker exec -it nginx_1129 /bin/bash`

### 2.2. config文件（## 注意upstream和server同级）

/etc/nginx/conf.d/default.conf

```xml
## 注意upstream和server同级
## 注意upstream和server同级
## 注意upstream和server同级
upstream redlock{
	server 192.168.1.95:8111 weight=1;
	server 192.168.1.95:8222 weight=1;
}	

server {
    listen       80;
    server_name localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
 

location / {
      #  root   /usr/share/nginx/html;
       # index  index.html index.htm;
        #  proxy_pass   http://nacos;
        proxy_pass http://redlock;
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
  location / {
      #  root   /usr/share/nginx/html;
       # index  index.html index.htm;
        #  proxy_pass   http://nacos;
        proxy_pass http://redlock;
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

```


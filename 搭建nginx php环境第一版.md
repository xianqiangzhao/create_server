# docker 安装nginx、php-fpm
## 构成图如下

![](https://github.com/xianqiangzhao/create_server/blob/master/img/photo1.png)
# 前提是安装好docker 并启动
# 建立nginx 的root目录
```
cd /usr/local/
mkdir  mynginx
cd  mynginx
mkdir html
```
# 建立php 脚本存放目录
```
cd /usr/local/mynginx
mkdir app
```
# 拉一个php-fpm
```
sudo docker run --rm   --name phpfpm   -v /usr/local/mynginx/app:/app  -dt   bitnami/php-fpm
```

# 拉一个nginx
```
sudo docker container run   --rm   --name mynginx   -v "$PWD/html":/usr/share/nginx/html   -p 0.0.0.0:80:80  -dt --link  phpfpm:fpm   nginx
```
--link 的意思是mynginx 容器的/etc/hosts 中增加一个fpm 的host，这样nginx.conf 中就可以配置.php 文件的请求转发给fpm容器了。
也就是容器互通的一种方法。

# copy nginx 配置路径
修改nginx 配置文件如果进入容易内还是比较麻烦的，直接copy 一份到本地，然后-v 建立关联。
```
docker container cp mynginx:/etc/nginx .
```
# 停止nginx
```
sudo docker container stop mynginx
```
# 修改nginx 配置文件，赋予处理.php 文件的能力
```
vi /usr/local/mynginx/conf/conf.d/default.conf
```
```
    location ~ \.php$ { 
        fastcgi_pass   fpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /app/$fastcgi_script_name;
        include        fastcgi_params;
    }
```
# 重新启动nginx 
```
sudo docker container run   --rm   --name mynginx   -v "$PWD/html":/usr/share/nginx/html  -v "$PWD/conf":/etc/nginx   -p 0.0.0.0:80:80  -dt    --link  phpfpm:fpm   nginx
```
# 确认docker 启动成功与否

docker ps
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
ccd9ecccf64e        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes       0.0.0.0:80->80/tcp   mynginx
a69200655d51        bitnami/php-fpm     "php-fpm -F --pid /o…"   43 minutes ago      Up 43 minutes       9000/tcp             phpfpm
```
# 写一个php脚本试一下吧
```
echo "<? phpinfo();" > /usr/local/mynginx/app/index.php
```
curl localhost/index.php

# 最后
 静态页面放到下
 /usr/local/mynginx/html
 动态页面放到下
 /usr/local/mynginx/app



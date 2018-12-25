
# 前提是安装好docker 并启动

cd /usr/local/

mkdir  mynginx
cd  mynginx
mkdir app        #php 脚本 路径

mkdir html       #nginx root 路径 

# 拉一个php-fpm
sudo docker run -dt --name phpfpm   -v /usr/local/mynginx/app:/app    bitnami/php-fpm

# 拉一个nginx
sudo docker container run   --rm   --name mynginx   -v "$PWD/html":/usr/share/nginx/html   　  -p 0.0.0.0:80:80  -dt    --link  phpfpm:fpm   nginx

# copy nginx 配置路径
docker container cp mynginx:/etc/nginx .
sudo docker container stop mynginx

# 修改nginx 配置文件
vi /usr/local/mynginx/conf/conf.d/default.conf


    location ~ \.php$ { 
        fastcgi_pass   fpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /app/$fastcgi_script_name;
        include        fastcgi_params;
    }

# 重新启动nginx 
sudo docker container run   --rm   --name mynginx   -v "$PWD/html":/usr/share/nginx/html  -v "$PWD/conf":/etc/nginx   -p 0.0.0.0:80:80  -dt    --link  phpfpm:fpm   nginx

# 确认docker 
docker ps

[rsa-key-20181224shotrong@instance-1 conf.d]$ sudo docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
ccd9ecccf64e        nginx               "nginx -g 'daemon of…"   10 minutes ago      Up 10 minutes       0.0.0.0:80->80/tcp   mynginx
a69200655d51        bitnami/php-fpm     "php-fpm -F --pid /o…"   43 minutes ago      Up 43 minutes       9000/tcp             phpfpm

# 写一个php脚本试一下

echo "<? phpinfo();" > /usr/local/mynginx/app/index.php

curl localhost

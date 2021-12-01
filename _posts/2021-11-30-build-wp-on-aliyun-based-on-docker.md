---
layout:     post
title:      基于docker构建wordpress的博客
subtitle:   centos7 aliyun
date:       2021-12-01
author:     就是我啦
header-img: img/post-bg-coffee.jpg
catalog: 	  true
tags:
    - docker    
    - blog  
    - 博客  
    - aliyun  
    - wordpress
---

### ☘ 搭好了，记录一下



> 双11买的阿里云机器，搭出来给孩子玩。
>
> 效果还不错
>
```
    8  docker ps
    9  yum install docker
   10  docker -v
   11  docker ps
   12  docker ps -a
   13  service docker start
   14  docker ps -a
   22  systemctl enable docker
  120  docker images
```

`123  docker import --message "OLDMysql" mysql.tar mysql:5.7`
`124  docker import --message "OLDwp" wp.tar wordpress `

```
  125  docker images
  152  docker exec -it 68a8e7aa6a7e /bash
  153  docker exec -it 68a8e7aa6a7e 
  154  docker ps -a
  155  docker exec -it 68a8e7aa6a7e  /bin/sh
  156  docker ps -a --no-trunc
  157  docker ps
  164  docker inspect --format='{{.LogPath}}' 68a8e7aa6a7e
  165  docker inspect 68a8e7aa6a7e
  166  docker inspect 68a8e7aa6a7e|grep log
  167  docker volume ls
  168  docker ps
  169  docker exec -it 68a8e7aa6a7e   /bin/sh
  170  docker ps
```
`192  docker run -d --privileged=true --name OLDMysql -e MYSQL_ROOT_PASSWORD=123456 -p 33306:3306 -v /root/mysql:/var/lib/mysql mysql:5.7 docker-entrypoint.sh mysqld`

`206  docker run -d --name OLDwp -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=123456 -e WORDPRESS_DB_NAME=myword -p 8080:80  -v /root/wp/html:/var/www/html --link OLDMysql:mysql wordpress docker-entrypoint.sh apache2-foreground`

```
  207  docker ps
  209  docker exec -it c40b4b523433 /bin/sh
  228  docker ps
  229  docker exec -it bfcc98706f8c /bin/sh
  231  docker ps
  232  docker exec -it c40b4b523433  /bin/sh
  243  docker ps
  248  docker ps
  250  docker export  bfcc98706f8c>mysql20211116.tar
  251  docker export c40b4b523433>wp20210016.tar
  254  docker ps
  255  docker ps -a
  257  history
```
`docker run -d --name OLDwp -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=123456 -e WORDPRESS_DB_NAME=myword -p 8080:80  -v /root/wp/html:/var/www/html --link OLDMysql:mysql wordpress docker-entrypoint.sh apache2-foreground`


`376  docker run -d --name RUIwp -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=123456 -e WORDPRESS_DB_NAME=myword2 -p 8082:80  -v /root/wp/html2:/var/www/html --link OLDMysql:mysql wordpress docker-entrypoint.sh apache2-foreground`

```
  378  docker exec -it bfcc98706f8c /bin/sh
  379  docker ps
```
`380  docker run -d --name LIUwp -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=123456 -e WORDPRESS_DB_NAME=myword3 -p 8083:80  -v /root/wp/html3:/var/www/html --link OLDMysql:mysql wordpress docker-entrypoint.sh apache2-foreground`


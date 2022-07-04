---

layout:     post
title:      mysql迁移到docker-linux
subtitle:   数据库迁移同时5.6升级到5.7
date:       2022-07-04
author:     就是我啦
header-img: img/post-bg-purplepalace.png
catalog: 	  true
tags:
    - mysql   
    - 数据库迁移    
    - docker
    - 数据库升级

---

# mysql迁移到docker-linux

> 本机上一直跑着windows版本的mysql，既然有了阿里云就想迁移到阿里云上去，顺便也练练docker

```sh
# linux版本mysql区分表名大小写，windows版本不区分！ 
```

### 起因

本机上一直跑着windows版本的mysql，既然有了阿里云就想迁移到阿里云上去，顺便也练练docker.

### 开干

说干就干。docker上的mysql镜像是现成的，版本5.7。如果拉最新的是8版本，就算了，还是用5.7吧。windows版本是5.6，差别小点，应该问题更少。

#### 第一步：启动docker容器

```shell
docker run -d --name erpmysql --privileged=true -v /root/mysqlerp:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=xxx -p 33307:3306 mysql:5.7 docker-entrypoint.sh mysqld  --lower_case_table_names=1 

```

这里要注意几点，一个是最好用-v参数来指定本机的mysql文件目录，这样即使容器废掉，也不会丢失数据库数据。

第二点是， ``docker-entrypoint.sh mysqld，``这部分是作为容器启动的参数。一般的镜像是不需要这个的，我这个镜像因为是import导入的，所以需要这个参数

第三点，``--lower_case_table_names=1 ``，这个参数就是为了指定Linux版本的mysql不要区分表名大小写。坑爹的参数。

#### 第二步：修改权限

mysql启动后，默认只能本机登录，可以用mysql命令登录进去，执行如下命令修改权限

```sql
grant all privileges on *.* to 'root'@'%' identified by 'xxx';

flush privileges;
```



#### 第三步：利用DBeaver工具迁移数据库

DBeaver工具刚用不久，不是很熟悉。本来它有导出数据库的功能，但经过实验，如果全库导出后（包含数据）再导入就报错：

![image-20220704175649749](/img/images/image-20220704175649749.png)

后来，不得已，改为只导出结构，导入的时候就不再报错。

只是这时候新数据库里面只有表结构，没有数据，再利用工具导入数据

![image-20220704175913004](/img/images/image-20220704175913004.png)

还可以一次导入多个表，很爽！

![image-20220704180048758](/img/images/image-20220704180048758.png)

至此，迁移完成！

有的时候存储过程和函数可能没有过来，那就单独再导一下就好。

中间有个问题，docker容器起不来，需要修改里面的参数，您可以使用`docker container cp`. 它适用于停止的容器。尝试

```shell
docker container cp [container]:/etc/mysql/my.cnf container-my.cnf
```

然后编辑 container-my.cnf 并复制回来：

```shell
docker container cp container-my.cnf [container]:/etc/mysql/my.cnf
```

```shell
docker cp :用于容器与主机之间的数据拷贝。

语法
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
OPTIONS说明：

-L :保持源目标中的链接

实例
将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

docker cp /www/runoob 96f7f14e99ab:/www/
将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。

docker cp /www/runoob 96f7f14e99ab:/www
将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

docker cp  96f7f14e99ab:/www /tmp/

```



### 总结

1. 这种事情知道能干，就是琐碎，溜溜搞了一天。
1. 遇到问题百度就好，就是有点烦人
1. 所以，经常记笔记是好习惯
1. 记住一些常用的docker命令，docker exec之类


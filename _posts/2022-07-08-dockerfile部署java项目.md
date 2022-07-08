---

layout:     post
title:      Dockerfile部署java项目
subtitle:   docker化是趋势
date:       2022-07-08
author:     就是我啦
header-img: img/post-bg-purplepalace.png
catalog: 	  true
tags:
    - Docker   
    - java    
    - jRE
    - alpine

---

# Dockerfile部署java项目

> 既然数据库都迁移到阿里了，索性连应用服务一起迁移到阿里
>
> 应用部署挺麻烦，干脆还是docker吧，快好省

```sh
# 其实以前操作部署过，这次就当温习一下 
```

### 起因

既然数据库都迁移到阿里了，索性连应用服务一起迁移到阿里。

使用docker，得先做好自己的镜像，一般以dockerfile开始。

当然，写dockerfile之前，java应用应该先打包，一般是zip包。

#### 第一步：准备dockerfile

```shell
# 环境
FROM  unifreq/alpine-jre8:latest
# 作者信息
MAINTAINER syj "329687273@qq.com"
# 将本地文件tmp挂载到容器
VOLUME /tmp
# 拷贝jar
ADD capms-release.zip /
# 设置暴露的端口号
EXPOSE 80
# 执行命令 
WORKDIR /              
#设定当前工作目录
RUN unzip capms-release.zip \
    && rm capms-release.zip
WORKDIR /capms

RUN chmod 777 start.sh
#RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#ENTRYPOINT ["/bin/bash", "java -version"] 
# ENTRYPOINT ["/capms/start.sh"] 
#ENTRYPOINT ["java","-jar","/eureka.jar"]
ENTRYPOINT ["/bin/sh", "-c", "java -Xverify:none -Xms256m -Xmx256m -Dundertow.port=80 -Dundertow.host=0.0.0.0 -cp ./config:./lib/*:./webapp/WEB-INF/lib/* com.capms.CapmsApp"]
#ENTRYPOINT ["java","-Xverify:none -Xms256m -Xmx256m -Dundertow.port=80 -Dundertow.host=0.0.0.0","-cp","./config:./lib/*:./webapp/WEB-INF/lib/*","com.capms.CapmsApp"]

```

使用现成的alpine-jre8镜像为底座，这个好处是体积小，整个镜像才100M+，不好处是只是JRE，如果需要jdk，还要自己补充。

其余的也简单，copy打包好的zip文件，解压就行了。

ENTRYPOINT，这个很重要，是镜像启动时的入口。要根据自己的情况写对。具体到本项目来说，可以运行start.sh文件脚本，也可以指定具体命令，我这里是指定了具体的命令。

#### 第二步：打包生成镜像

直接打包就好

```shell
docker build -t capms:1.0 .
```



#### 第三步：利用镜像生成运行容器

```shell
docker run -d -p 8888:80 capms:1.0
```

经过这三步，java应用应该就可以正常运行起来了。

后面根据需要再决定是否用nginx进行一下反向代理。

### 总结

1. dockerfile还算好理解
1. 和数据库镜像一起启动的话，还可以考虑docker-compose
1. alpine-jre8很好用，就是只是jre，如果非要用jdk的话，可以看看其他jdk镜像，比如dragonwell
1. docker前两年就嚷嚷着要过时，更多的是商业的原因吧，这么好的东西，总有生命力


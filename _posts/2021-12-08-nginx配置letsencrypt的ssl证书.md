---
layout:     post
title:      nginx配置letsencrypt的ssl证书
subtitle:   nginx https配置
date:       2021-12-08
author:     就是我啦
header-img: img/post-bg-alibaba.jpg
catalog: 	  true
tags:
    - nginx    
    - https  
    - ssl      
    - letsencrypt
---

# nginx配置letsencrypt的ssl证书

今天想把https配上，避免烦人的chrome不安全提示。

阿里云以前提供免费的ssl证书，结果打开，发现又变了，只限20个，而且还按自然年重置云云，现在12月，按自然年岂不是只能用几天？索性作罢。阿里云的服务真是越来越贵了

## 1. let's encrypt

想起来以前用过免费的letsencrypt，好像还很好用。

翻以前的笔记，找不到了。

没办法就只好百度。一堆文章，结果没有一个能用的。

最后，还是官网上的说明靠谱。



官网最后链接到了这里，

[cerbot](https://certbot.eff.org/instructions?ws=nginx&os=centosrhel7)

按照说明进行操作就好了，简直傻瓜化。

这是它自动增加的nginx配置



```nginx
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/gummi.top/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/gummi.top/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
```


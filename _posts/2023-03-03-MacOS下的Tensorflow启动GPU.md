---
layout:     post
title:      MacOS下的Tensorflow启动GPU
subtitle:   黑苹果里用GPU玩AI
date:       2023-03-03
author:     就是我啦
header-img: img/post-bg-map.jpg
catalog: 	  true
tags:
    - AI   
    - 人工智能  
    - Tensorflow
    - Keras    
    - GPU

---

# MacOS下的Tensorflow启动GPU

> 显卡厂商这几年不是一般的春风得意，简直是顺风顺水，老天眷顾。先是前几年的矿潮狂赚，这两年矿潮港想过去，AI又接了上来。老黄简直命太好。
>
> 同样的AI计算，GPU比CPU还是效率高得多。在大数据集情况下，CPU 版本无法加速运算，计算速度相对缓慢，此时，GPU的性能要比CPU强大很多，所以推荐使用GPU。但在小数据集的情况下CPU和UGPU的性能差别不大。CPU 版本暂可用作学习，如为了学习模型算法，数据集不大，使用 CPU 版本也能勉强应付。
>
> - 黑苹果的显卡这么好，不用一下太可惜啊

```sh
事实证明，GPU大数据集的性能比CPU好得多
```

## 缘起

自从装好了黑苹果，基本上开发都是在黑苹果上做。确实黑苹果开发有很多方便的地方，brew的安装方式，各种IDE照样兼容，还有parellels desktop可以方便的安装各种虚拟机。就是有点遗憾，因为显卡是AMD的570，在windows下的Tensorflow keras程序用的是N系显卡的CUDA，在MacOS下既不是N系显卡也不支持CUDA。正感遗憾的时候偶尔搜到了MacOS下有更方便的GPU启用方式，就立马试试。

结果发现，真香！😄。

## intel版本的安装
官方教程
https://developer.apple.com/metal/tensorflow-plugin/

### 1.安装虚拟环境
在这里我使用conda，也可以用其他的

```conda create -n tf-metal-2.6 python=3.8```

### 2.激活虚拟环境
```conda activate tf-metal-2.6```

### 3.安装tensorflow-macos和tensorflow-metal 
```pip install tensorflow-macos tensorflow-metal```
如果你是macOS12，可能识别不到你的型号，报错not a supported wheel on this platform

前面加上SYSTEM_VERSION_COMPAT=0 

```shell
SYSTEM_VERSION_COMPAT=0 pip install tensorflow-macos tensorflow-metal
```


至此，大功告成！！

## 测试

赶紧用程序试了试，真好使！

```python
import tensorflow as tf
import timeit
import numpy as np
import matplotlib.pyplot as plt
def cpu_run(num):
  with tf.device('/cpu:0'):
    cpu_a=tf.random.normal([1,num])
    cpu_b=tf.random.normal([num,1])
    c=tf.matmul(cpu_a,cpu_b)
  return c
def gpu_run(num):
  with tf.device('/gpu:0'):
    gpu_a=tf.random.normal([1,num])
    gpu_b=tf.random.normal([num,1])
    c=tf.matmul(gpu_a,gpu_b)
  return c
k=10
m=7
cpu_result=np.arange(m,dtype=np.float32)
gpu_result=np.arange(m,dtype=np.float32)
x_time=np.arange(m)
for i in range(m):
  k=k*10
  x_time[i]=k
  cpu_str='cpu_run('+str(k)+')'
  gpu_str='gpu_run('+str(k)+')'
  #print(cpu_str)
  cpu_time=timeit.timeit(cpu_str,'from __main__ import cpu_run',number=10)
  gpu_time=timeit.timeit(gpu_str,'from __main__ import gpu_run',number=10)
  # 正式计算10次，取平均时间
  cpu_time=timeit.timeit(cpu_str,'from __main__ import cpu_run',number=10)
  gpu_time=timeit.timeit(gpu_str,'from __main__ import gpu_run',number=10)
  cpu_result[i]=cpu_time
  gpu_result[i]=gpu_time
print(cpu_result)
print(gpu_result)
fig, ax = plt.subplots()
ax.set_xscale("log")
ax.set_adjustable("datalim")
ax.plot(x_time,cpu_result)
ax.plot(x_time,gpu_result)
ax.grid()
plt.draw()
plt.show()

```



能明显看出CPU和GPU的性能区别：

![image-20230303145651062](/Users/shenyaojun/work/shenyaojun.github.io/img/images/image-20230303145651062.png)

蓝线是CPU耗时，红线是GPU。很明显，大数据量下GPU性能高的多。

运行的时候还会给出显卡使用的提示，看着满满的满足感：

```shell
Metal device set to: AMD Radeon RX 570

systemMemory: 64.00 GB
maxCacheSize: 4.00 GB
```



### 总结

1. GPU大数据集的性能比CPU好得多
1. MacOS做开发还是很方便
1. 黑苹果无线上网不完美，还得连手机才能上，有点儿忍不住想花钱上无线WIFI了

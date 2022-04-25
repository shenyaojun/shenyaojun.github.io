---
layout:     post
title:      自动化测试框架RobotFramework
subtitle:   web端UI的测试自动化
date:       2022-04-25
author:     就是我啦
header-img: img/post-bg-rwd.jpg
catalog: 	  true
tags:
    - 自动化测试框架    
    - 自动化测试        
    - RobotFramework      
    - Python
    - Selenium   
---

# 自动化测试框架RobotFramework

> 自动化测试框架已经发展的比较成熟了
>
> 重复的劳动就可以考虑自动化，IT界不变的法则

```yacas
试验一下新的文章块，只要把语言设成yacas
比较醒目
数字也好看：
1. 多用用就好了
2. 学无止境
```

这些天侧重学了一下有关自动化测试的知识，拓展了一下知识面。感触很多，参考资料主要有：

* https://www.bilibili.com/video/BV1a64y1r7AT?p=2
* https://www.bilibili.com/video/BV1QM4y137xX?p=145&spm_id_from=pageDriver
* https://www.bilibili.com/video/BV1pK4y1H7LH/?p=2&spm_id_from=pageDriver
* https://www.bilibili.com/video/BV1du411o7DH?p=5&spm_id_from=pageDriver
* https://www.bilibili.com/video/BV1kK4y1Y7Q3?p=6

最后发现RobotFramework是个比较好用的工具。当然，其他的自动化测试框架并不了解，所以，说不定还有其他更好用的



## 1. 首先，什么是自动化测试？

自动化测试是把``以人为驱动的测试行为``转化为``机器执行``的一种过程。通常，在设计了测试用例并通过评审之后，由测试人员根据测试用例中描述的规程一步步执行测试，得到实际结果与期望结果的比较。在此过程中，``为了节省人力、时间或硬件资源，提高测试效率，``便引入了自动化测试的概念。

> 测试自动化可以在已经存在的正式测试过程中自动化一些重复但必要的任务，或者添加额外的难于手工执行的测试。



## 2. 自动化测试主要有哪些？

```yacas
自动化测试一般分三种：1.单元自动化测试，2.接口自动化测试，3.UI自动化测试。

其中单元自动化测试一般由研发人员自己进行测试，测试人员主要进行接口以及UI的自 动化测试，但是由于UI的需求变化比较频繁，所以接口测试是测试人员做的最多的
```



## 3. 自动化测试的优势

主要具备以下优势：

（1）回归测试更方便可靠，可运行更多、更繁琐的测试，且快速高效；

（2）可执行一些手工测试执行相当困难或者做不到的测试，如大量的用户并发；

（3）可以更好的利用资源，具有一致性和可重复性的特点，自动化测试脚本完全可复用；

（4）提升了软件的可信度；

（5）可以多环境下测试等。



## 4. 自动化测试的劣势

（1）``永远不可能完全替代手工测试``。自动化测试``无法做到手工测试的覆盖率``，不是每个测试用例都适合实行自动化。

（2）``手工测试发现的bug远比自动化测试多。自动化测试几乎是无法发现新bug的，最大的用途是用来回归``，确保曾经的bug没有在新的版本上重新出现。

（3）自动化测试工具比较死板，灵活性比较差。自动化测试的效果好坏，完全取决于测试工程师。

（4）成本投入大，风险高。对测试人员的技术要求高，对测试工具同样也高。

（5）测试用例需要根据版本迭代进行更新，有一定的维护成本

（6）自动化测试的``产出价值往往在于长期的回归测试``，短期内发挥的作用可能不明显。





## 5. 实操

以前安装过Anaconda，所有Python3的安装过程就省略了。但不排除可能引起问题



Ride的安装，要去掉中文乱码：

https://blog.csdn.net/qq_42534619/article/details/118255843

```yacas
RIDE输出乱码问题解决
1、问题:RIDE自动测试case在ride界面控制台中输出乱码,在py的ide中可以正常输出中文
2、解决方案:修改RIDE的配置文件，找到{testrunnerplugin.py}这个文件，在python安装目前下，我的安装参考路径如下：D:\Program Files\python\Lib\site-packages\robotide\contrib\testrunner\testrunnerplugin.py

3、修改这个py文件，找到下面的代码：

encoding = {‘CONSOLE’: CONSOLE_ENCODING,
‘SYSTEM’: SYSTEM_ENCODING,
‘OUTPUT’: OUTPUT_ENCODING}　　　　
将【SYSTEM_ENCODING】为【OUTPUT_ENCODING】 重启即可；目的使系统输出也作为上级输出即py输出.
```



WebDriver的版本一定要对应，和python的执行文件放在同一路径下

![image-20220425153055594](/img/images/image-20220425153055594.png)

### 总结

- python

- unittest，pytest

- selenium

- webdriver，chromedriver

- xpath

- RobotFramework

- allure

- appium

- postman

- jmeter

- ZTF

- jenkins

- 收获颇丰，一个测试自动化有这么多知识点

  

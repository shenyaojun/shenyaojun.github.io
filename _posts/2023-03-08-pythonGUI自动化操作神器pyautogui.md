---
layout:     post
title:      PythonGUI自动化操作神器pyautogui
subtitle:   又一个python自动化的神器
date:       2023-03-08
author:     就是我啦
header-img: img/post-bg-balloons.jpg
catalog: 	  true
tags:
    - 自动化   
    - Python  
    - Pyautogui
    - Selenium    
    - RobotFramework

---

# PythonGUI自动化操作神器pyautogui

> 今天偶然刷到PythonGUI的介绍视频，觉得挺好，记下来

```sh
Python的自动化测试包有很多，除了pyautogui还有pywinauto、win32gui等。相对pyautogui用的更多一些，直接通过屏幕定位来操作鼠标和键盘，不用抓取窗口句柄等结构，简单粗暴方便
```

## 缘起

今天B站上偶尔看到了这个视频：https://www.bilibili.com/video/BV1844y1N7xH

去年研究过一阵自动化测试框架（看[这里](https://syj.gummi.top/2022/04/25/%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95%E6%A1%86%E6%9E%B6RF/)），基本都是python下的，像selenium、webdrive、RobotFramework等技术。这些技术都很好用，但是基本都局限于Web自动化测试，侧重于通过CSS或者HTML语法的解析，识别其对象，然后进行操作。而对于非Web类应用，就有点力不从心了。Pyautogui则不同，其对操作对象的识别可以通过```图像对比```，一下子方便了很多。

pyautogui是用来做GUI桌面应用自动化的Python包，功能类似于按键精灵：可以实现控制鼠标、键盘、消息框、截图、定位功能。

这也就意味着Pyautogui不仅可以做web自动化，而是对于所有有图形显示界面的程序，都可以做自动化了。比如说，游戏等。

## 一个自动登录数据库的例子
这里有个登录PL/SQL的例子：

```python
# -*- coding: utf-8 -*-
"""
Created on Tue Apr 13 15:09:40 2021

@author: yunyi.wang
"""


import pyautogui
import time

# 回到桌面
pyautogui.hotkey('win','d')
time.sleep(0.5)

# 打开数据库
coords = pyautogui.locateOnScreen(r'C:\Users\yunyi.wang\Desktop\file\Kingstar_files\自动化报表RPA需求\PLSQL.png')
pyautogui.click(pyautogui.center(coords), clicks=2)

time.sleep(6)

# 定位用户名
coords = pyautogui.locateOnScreen(r'C:\Users\yunyi.wang\Desktop\file\Kingstar_files\自动化报表RPA需求\username.png')
pyautogui.click(pyautogui.center(coords), clicks=1)
pyautogui.hotkey('ctrlleft','a')
pyautogui.press('shiftleft')
pyautogui.typewrite(message='IM12',interval=0.1)
time.sleep(0.5)
# 定位密码
coords = pyautogui.locateOnScreen(r'C:\Users\yunyi.wang\Desktop\file\Kingstar_files\自动化报表RPA需求\password.png')
pyautogui.click(pyautogui.center(coords), clicks=1)
pyautogui.hotkey('alt','a')
pyautogui.typewrite(message='wolf123',interval=0.1)
time.sleep(0.5)
pyautogui.press('enter')

time.sleep(3)

# 新建SQL窗口
coords = pyautogui.locateOnScreen(r'C:\Users\yunyi.wang\Desktop\file\Kingstar_files\自动化报表RPA需求\PLSQL_new.png')
pyautogui.click(pyautogui.center(coords), clicks=1)
coords = pyautogui.locateOnScreen(r'C:\Users\yunyi.wang\Desktop\file\Kingstar_files\自动化报表RPA需求\PLSQL_SQLwin.png')
pyautogui.click(pyautogui.center(coords), clicks=1)
# 输入SQL语句，完成查询
pyautogui.typewrite(message='select * from IM12.ASHAREEODDERIVATIVEINDICATOR',interval=0.05)
pyautogui.press('F8')


```



## 替代selenium进行web自动化测试

还有一个例子，演示替代selenium进行web自动化测试。不过，并不建议使用pyautogui进行web自动化测试，因为```元素的ui一旦有长宽变化，或者风格的变化，执行时就会发生异常```，仅当学习使用。

以使用selenium打开百度，并在输入框输入“只宅不技术”，之后点击搜索为例

代码如下：

```python
#coding=utf-8
from selenium import webdriver
import time
#打开火狐浏览器
driver=webdriver.Firefox()
#打开百度
driver.get("https://www.baidu.com")
time.sleep(2)
#找到输入框输入 只宅不技术
driver.find_element_by_id("kw").send_keys(u'只宅不技术')
#点击搜索框
driver.find_element_by_id("su").click()

```

若要使用pyautogui替代selenium，需要先进行截图，然后通过图像识别操作

首先需要利用截图工具进行截图，比如QQ就可以进行截图，需要截的图片有

1、火狐浏览器的图标，将其命名为firefox.png

2、输入url的地址框，将其命名为url.png

3、进行搜索的输入框，将其命名为kw.png

4、进行搜索的搜索按钮，将其命名为su.png

由于typewrite()函数无法输入中文，所以事先把“只宅不技术”复制到了粘贴板，输入时候粘贴一下就行，将截图与代码放置在同一路径下【需要注意，整个屏幕上只能有一个火狐的图标，不然会报错】

![img](https://img-blog.csdnimg.cn/37cb3226cd8147428e35105d4700b491.png)

代码如下：

```python
import pyautogui
import time

#定义图像识别双击事件
def mouseDoubleClick(image):
    x,y=pyautogui.locateCenterOnScreen(image)
    pyautogui.click(x,y,clicks=2,interval=0.2,duration=0.2,button='left')

#定义单击事件
def mouseClick(image):
    x,y=pyautogui.locateCenterOnScreen(image)
    pyautogui.click(x,y,clicks=1,interval=0.2,duration=0.2,button='left')

#双击火狐浏览器的图标
mouseDoubleClick(image='firefox.png')
time.sleep(3)
#双击浏览器的url地址框
mouseClick(image='url.png')
#在地址框输入百度地址，然后回车
pyautogui.typewrite('www.baidu.com')
pyautogui.keyDown('enter')
pyautogui.keyUp('enter')
time.sleep(2)
#双击搜索框
mouseClick(image='kw.png')
#将只宅不技术粘贴到搜索框
pyautogui.hotkey('ctrl','v')
time.sleep(2)
#点击搜索
mouseClick(image='su.png')

```

### 可扩展自动化脚本

还有一个B站up主用pyautogui写了一个可扩展的自动化脚本。具体可以看[这个](https://www.bilibili.com/video/BV1T34y1o73U)。

代码已经拿下来，还是有点意思。

### 总结

1. Pyautogui是用来做GUI桌面应用自动化的Python包
1. Pyautogui基于图像识别做元素定位，所以如果显示效果有变化，就识别不了
1. Pyautogui可以作为selenium的补充，一起做Web自动化
1. 黑苹果无线上网不完美，没忍住还是花钱上无线WIFI了。现在蓝牙还不行。。。

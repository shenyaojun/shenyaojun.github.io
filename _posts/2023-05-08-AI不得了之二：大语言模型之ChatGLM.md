---
layout:     post
title:      AI不得了之二：大语言模型之ChatGLM
subtitle:   AI让电脑彻底进入智能时代
date:       2023-05-08
author:     就是我啦
header-img: img/post-bg-keybord.jpg
catalog: 	  true
tags:
    - ChatGPT   
    - ChatGLM
    - AI
    - 人工智能
    - LLM    
    - 大语言模型    
    - AIGC

---

# AI不得了之二：大语言模型之ChatGLM

> ChatGPT的出现突然震惊了世人，个人感觉，这标志着IT时代进入了新的阶段，彻底进入了智能时代。
>
> 国内的大厂跟进是必然的，百度的文心一言，阿里的通义千问，华为的盘古等等，都唯恐赶不上趟。飞机马上起飞，再不登机，可能就要被永远拉下了。

```sh
以前的IT只能叫前智能时代，其标志就是人和电脑的沟通还不能自如。为了和电脑沟通，让电脑理解人的意图，不得不借助于
其他工具，让人的行为降级，以适应电脑的傻。比如，鼠标的发明、键盘的发明，都是人为了适应电脑的傻，其实非常的不人
性化。真正的智能，应该像和人沟通一样，我说你就能听明白。现在，ChatGPT的出现，终于让这一愿景接近实现了。

```

## 引子

GPT4的火爆惊醒了一向以为我们在AI上已经和美国并驾齐驱的国人。原来，我们还差这么远。

知耻而后勇总是好的，好在一些国产的大模型也在努力追赶，包括：

* 百度，文心一言
* 阿里，通义千问
* 商汤，日日新
* 华为，盘古
* 腾讯，混元
* 科大讯飞，星火，前天（5.6）刚推出

太多了，还有很多开源的大模型，个个都号称能接近GPT4：

* GPT2。只有2还开源，从3开始就是闭源了，开启了订阅付费模式
* Meta：LLaMA【大羊驼】。 几周前，MetaAI推出了大语言模型LLaMA，其不同版本包括70亿、130亿、330亿和650亿的参数，虽然比GPT3还小，但LLaMA在许多任务上的性能都能够追平甚至超越GPT3。而有趣的是，LLaMA起初并未开源，但在发布后不久，模型便在4chan论坛上泄露了。然而这个乌龙事件却也使LLaMA因祸得福，在“被迫”开源后，LLaMA引发了大量开发者的关注，基于其构建的模型也如雨后春笋般诞生。这场本应是不幸的事情也使得LLaMA成为了如今LLM领域最具影响力的创新来源之一。
* 斯坦福大学：**Alpaca**【羊驼】。Alpaca是斯坦福大学通过Meta的LLaMA 70亿微调而成的全新模型，仅使用了52k数据，但其性能却基本达到了GPT3.5的水平。而Alpaca的关键优势则在于低到出奇的训练成本——仅需不到600美元。
* Databricks：Dolly2.0，全开源可商用
* Hugging Face：BLOOM，体量惊人，号称拥有1760亿参数
* 阿卜杜拉国王科技大学**MiniGPT4**，图像对话能力可观
* Stability AI：StableLM，万亿token训练。这个就是Stable Diffusion那家公司的了，值得期待
* **清华大学``ChatGLM``**，开辟小而精方向。ChatGLM是由清华技术成果转化的公司智谱AI开发的开源、支持中英双语的对话语言模型，基于General Language Model (GLM) 架构研发，**拥有62亿参数**，支持在单张消费级显卡上进行推理使用，在保障平民消费能力的情况下，具有小而精的特点。但由于模型容量较小，ChatGLM也不可避免的存在着模型记忆和语言能力较弱;可能产生错误内容和多轮对话能力不足等一些缺点。
* UC伯克利：**Vicuna**【小羊驼】，上个月刚开源的。据说GPT-4评测过几个大模型，最终的打分结果显示，**在130亿参数LLaMA基础上微调而来的Vicuna，达到了ChatGPT（GPT-3.5）性能的90%**，超过了LLaMA-13B本身和斯坦福的Alpaca
* 复旦好像还有个MOSS



## 什么是wenda

总的来说，清华大学的ChatGLM表现还是不错的，据官方介绍，ChatGLM参考了ChatGPT的设计思路，在千亿基座模型**GLM-130B**中注入了代码预训练，通过有监督微调等技术来实现人类意图对齐（就是让机器的回答符合人类价值观、人类期望）。这个GLM-130B的来头值得说道一番。

它是由清华大学**知识工程实验室**(KEG)与**智谱AI**共同研发的一个大规模中英文预训练语言模型，参数**1300亿**，去年8月正式对外发布。

不同于BERT、GPT-3以及T5的架构，GLM-130B是一个包含**多目标函数的自回归预训练模型**。

就在GPT4发布的同一天，清华大学**唐杰团队**在微博宣布：

基于千亿参数大模型的**对话机器人ChatGLM**，现在开启邀请制内测。`chatGLM-6B`是GLM-130B的**缩小版**，虽然只有62亿参数，但基本达到了可以玩一下的水平。尤其是，还比较谦虚，这是它对自己和ChatGPT相比的回答：

![image-20230508163657223](\img\images\image-20230508163657223.png)



模型有了，怎么玩和怎么用也是问题。有国内大神做了个开源项目：wenda，可以解决这个问题。这是它的github地址：https://github.com/l15y/wenda/



# 闻达：一个大规模语言模型调用平台

Wenda采用**知识库+auto脚本**的形式为LLM提供**生成能力**，充分考虑个人和中小企业的资源问题，以及国内大背景下知识安全和私密性问题,实现使小模型获得近似于大模型的生成能力。

1. 目前支持模型：`chatGLM-6B`、`chatRWKV`、`chatYuan`、`llama系列`以及`openaiapi`和`chatglm130b api`，初步支持`moss`。
2. 使用知识库扩展模型所知信息，使用auto提高模型生成质量和复杂问题解决能力
3. 支持`chatGLM-6B`、`chatRWKV`、`llama系列`流式输出和输出过程中中断
4. 自动保存对话历史至浏览器（多用户同时使用不会冲突，`chatRWKV`历史消息实现方式需使用string）
5. 对话历史管理（删除单条、清空）
6. 支持局域网、内网部署和多用户同时使用。

我试了试，在我本机1660Ti 6G显存下可以跑起来wenda+chatGLM-6B。结合fess的话，还可以处理本地知识库。

![image-20230508164531728](\img\images\image-20230508164531728.png)

![image-20230508164554291](\img\images\image-20230508164554291.png)

不过，测试的时候也发现它对本地知识库的理解能力还有待提高。本地知识库的文档也要写的很详细才行。如果太简略，其理解可能有问题。

## 编程能力测试

测了一下编程的问题，发现基本处于能用的水平。比如，我问：write a program in python to download a short video from bilibili，ChatGLM很快就能回答：

```python
Sure, here's a sample program in Python to download a short video from Bilibili:


import requests

from bs4 import BeautifulSoup


# Set the URL of the video to be downloaded

url = "https://www.bilibili.com/video/ID_123456"


# 发送 GET 请求以获取视频内容

response = requests.get(url)


# 解析 HTML 页面

soup = BeautifulSoup(response.text, "html.parser")


# 获取视频标题和时长

title = soup.find("title").get_text()

duration = soup.find("video/info").find("meta").get_text()


# 下载视频

with open("download.mp4", "wb") as f:
    
    for link in title, duration:
        
        f.write(link)
        
This program first sets the URL of the video to be downloaded, and then uses the requests
library to send a GET request to the Bilibili website. The response from the request is 
then解析d using BeautifulSoup, and the video title and length are extracted. Finally, the
video is下载 using the with open() statement.


You can replace the id_123456 with the actual video ID of the short video you want to 
download and the https://www.bilibili.com/video/ID_XXXX with the URL of the video you 
    want to download.
    

Note that this is just a simple example, and it may not work as-is on all Bilibili 
websites. Additionally, the Bilibili website may have different privacy policies and
terms of use, so you may need to comply with them to download the video.



```

回答的英文里面还夹杂这中文，有点疑惑。不过，总体看，回答的还是像模像样，有点那个意思了。当然，我拿这个程序真的跑了一下，肯定是失败的。B站有很强的防范能力，想下载视频不是那么容易的。



## 关于ChatRWKV

ChatRWKV是一个国人做的开源项目，是对标ChatGPT的，基于RWKV (100% RNN) 语言模型的类似ChatGPT 的开源工具。该开源项目支持问话对答式交流，而且对中国网文写作方面的能利更好，一般主要用来写网文或者小说。这是它的github: https://github.com/BlinkDL/ChatRWKV

目前主要有14B和7B模型：

**Raven 14B** (finetuned on Alpaca+ShareGPT+...) Demo: https://huggingface.co/spaces/BlinkDL/ChatRWKV-gradio

**Raven 7B** (finetuned on Alpaca+ShareGPT+...) Demo: https://huggingface.co/spaces/BlinkDL/Raven-RWKV-7B

在我的1660Ti 6G显卡下就能玩，让它写了两千字的小说，结果，开个头就跑题了：）



### 其他

#### 1. text2video

基于多阶段文本到视频生成扩散模型, 输入描述文本，返回符合文本描述的视频，出自阿里达摩院。

文本到视频生成扩散模型由文本特征提取、文本特征到视频隐空间扩散模型、视频隐空间到视频视觉空间这3个子网络组成，整体模型参数约17亿。支持英文输入。扩散模型采用Unet3D结构，通过从纯高斯噪声视频中，迭代去噪的过程，实现视频生成的功能。

https://huggingface.co/damo-vilab/text-to-video-ms-1.7b

试用了一下，能玩，但是，离Stable Diffusion真正实用的水平还比较远。

#### 2. Claude

Claude 由OpenAI前副总裁离职创立，和ChatGPT同宗。

现在国内已经不能用了，可用地区里没有中国。

#### 3. text2audio bark

开源地址：[GitHub - suno-ai/bark: Text-Prompted Generative Audio Model](https://link.zhihu.com/?target=https%3A//github.com/suno-ai/bark)

Bark是一个由 Suno 创建的基于变压器的文本-音频模型。Bark可以生成高度逼真的多语言语音以及其他音频--包括音乐、背景噪音和简单的声音效果。该模型还可以产生非语言交流，如笑、叹气和哭泣。为了支持研究界，我们正在提供访问预训练的模型检查点，准备进行推理。

简单来说，这就是一个 AI 文字生成语文模型，最大的亮点就是可以生成各种音效，这是传统文字转语音工具做不到的。

试听效果：[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://link.zhihu.com/?target=https%3A//suno-ai.notion.site/Bark-Examples-5edae8b02a604b54a42244ba45ebc2e2)

官方提供的 huggingface 体验地址：[Bark - a Hugging Face Space by suno](https://link.zhihu.com/?target=https%3A//huggingface.co/spaces/suno/bark)

就是速度太慢了！

试用后好像对中文支持不太好，一股外国人说中文的味道。



最后，感谢**百度飞桨**，免费学习平台，真的不错！



### 总结

1. 以ChatGPT的出现为标志，人类彻底进入了智能时代
1. 大模型目前是GPT4一骑绝尘，其他大模型百花齐放。应该是人类之福
1. ChatGLM如果再强大一点，真的可以支持企业知识库商用了
1. 看国产开源的大模型，离“贾维斯”还有点远。不知道大厂那几个会怎么样
1. 憧憬着吧

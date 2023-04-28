---
layout:     post
title:      AI不得了之一：GFPGAN和Stable Diffusion
subtitle:   AI让电脑彻底进入智能时代
date:       2023-04-28
author:     就是我啦
header-img: img/post-bg-map.jpg
catalog: 	  true
tags:
    - GFPGAN   
    - Stable Diffusion
    - AI
    - 人工智能
    - LLM    
    - 大语言模型    
    - AIGC

---

# AI不得了之一：GFPGAN和Stable Diffusion

> ChatGPT的出现突然震惊了世人，个人感觉，这标志着IT时代进入了新的阶段，彻底进入了智能时代。
>
> 国内的大厂跟进是必然的，百度的文心一言，阿里的通义千问，华为的盘古等等，都唯恐赶不上趟。飞机马上起飞，再不登机，可能就要被永远拉下了。

```sh
以前的IT只能叫前智能时代，其标志就是人和电脑的沟通还不能自如。为了和电脑沟通，让电脑理解人的意图，不得不借助于其他工具，让人的行为降级，以适应电脑的傻。比如，鼠标的发明、键盘的发明，都是人为了适应电脑的傻，其实非常的不人性化。真正的智能，应该像和人沟通一样，我说你就能听明白。现在，ChatGPT的出现，终于让这一愿景接近实现了。
```

## 引子

一切开始于GFPGAN。姥姥已过世多年，很遗憾的是姥姥留下的照片不多，有一天偶尔想，电脑不是能让照片变清晰吗，干嘛不试试。于是，就找到了GFPGAN。

之所以是GFPGAN，是因为它是国产的，由腾讯的团队开发。这是它的[Github网址](https://github.com/TencentARC/GFPGAN)。

目前，腾讯开源的这个GFP-GAN项目有几种体验方式，比如Colab、Hugging Face或本地运行代码测试体验。第一个咱就不说了，吃了腿短的亏，没梯子翻不了墙。第二个是在线运行的，提供一个demo，这个理论上是可以玩的。

不过，玩了之后才发现，GFPGAN主要做``脸部增强``，要是想做整个照片的清晰度增强，需要作者的另一个项目：[Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN)。

## 什么是GAN
理解GFPGAN，得先理解GAN，GAN的全称是 ``Generative Adversarial Network``，中文是**生成对抗网络**。自从2014年被``Ian Goodfellow``提出以来，掀起来了一股研究热潮。

GAN是一种深度学习模型，是近年来复杂分布上**无监督学习**最具前景的方法之一。模型通过框架中（至少）两个模块：生成模型（Generative Model）和判别模型（Discriminative Model）的互相博弈学习产生相当好的输出。原始 GAN 理论中，并不要求 G 和 D 都是神经网络，只需要是能拟合相应生成和判别的函数即可。但实用中一般均使用深度神经网络作为 G 和 D 。一个优秀的GAN应用需要有良好的训练方法，否则可能由于神经网络模型的自由性而导致输出不理想。（来自百度百科）。

其用途主要有：

* 图像生成：GAN最常使用的地方就是图像生成，如超分辨率任务，语义分割等等。
* 数据增强：GAN生成的图像来做数据增强，使用对抗生成网络生成虚假数据来扩充数据集。

其最大的特点是为深度网络提供了一种**对抗训练**的方式，此方式有助于解决一些普通训练方式不容易解决的问题。并且``Yan lecun``明确表示GAN是近几十年除了面包机最伟大的发明，并且希望是自己发明的GAN。

其原理可以用这个图来表示：

![img](\img\images\f1a2c2f0025761f3f4b75ca9cc25501c.png)



GAN 包含了两个神经网络，**生成器**（generator）和**辨别器**（discriminator），两者互相博弈不断变强，即生成器产出的东西越来越逼真，辨别器的识别能力越来越牛逼。

生成器和辨别器之间的关系很像**造假者**（counterfeiter）和**鉴定者**（Appraiser）之间的关系。

* 造假者不断造出假货，目的就是蒙骗鉴定者，在此过程中其造假能力越来越高。

* 鉴定者不断检验假货，目的就是识破造假者，在此过程中其鉴定能力越来越高。

GAN 是造假者的，也是鉴定者的，但归根结底还是造假者的。GAN 的最终目标是训练出一个**“完美”**的造假者，即能让生成让鉴定者都蒙圈的产品。

一动图胜千言，下图展示“**造假者**如何一步步生成逼真的蒙娜丽莎画而最终欺骗了**鉴定者**”的过程。

![img](\img\images\97327e564c19f269a6921ac88e69ef83.gif)



在此过程中，每当造假者生成一幅图。鉴定者会给出反馈，造假者从中学到如何改进来画出一张逼真图。

回到神经网络，**造假者**用**生成器**来建模，**鉴定者**用**辨别器**来建模。

![img](\img\images\f0b27a0b013082a1f92c297a418bae12.gif)



## GFPGAN

GFPGAN的程序本地跑起来还是比较容易的，按照官方的文档做就行。当然，硬件条件是最好有显卡，还最好是N系的。

这是我在本地跑的一个结果：

![test1_00](\img\images\test1_00.png)

可以看出还是挺成功的。不过，这个原图的模糊程度并不严重。太严重的话，还是会有不少问题，比如，它经常把我给P成双眼皮：）。

### Stable Diffusion

玩了下GFPGAN之后，对电脑生成图像产生兴趣，自然就看到了stable diffusion。

目前AI绘画最火的当属**Midjorney和Stable Diffusion**，但是由于Midjourney没有开源，因此国内主要玩的是Stable Diffusion，而且这个门槛不高，普通的带显卡电脑就能玩儿，实在没显卡，用CPU也行，就是你得等得起。加上国内有秋叶大佬作的懒人包，大有星火燎原的趋势了。AI作画近期取得如此巨大进展的原因个人认为有很大的功劳归属于Stable Diffusion的开源。

公开资料显示，**Stable** Diffusion**是StabilityAI公司于2022年提出的**，[论文](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2112.10752.pdf)和[代码](https://link.zhihu.com/?target=https%3A//github.com/CompVis/stable-diffusion)都已开源。StabilityAI在10月28日完成了1.01亿美元的融资，目前估值已经超过10亿美元。

#### Stable Diffusion的原理

Stable diffusion是一个基于**Latent Diffusion Models（潜在扩散模型，LDMs）**的**文图生成**（text-to-image）模型。具体来说，得益于Stability AI的计算资源支持和LAION的数据资源支持，Stable Diffusion在[LAION-5B](https://link.zhihu.com/?target=https%3A//laion.ai/blog/laion-5b/)的一个子集上训练了一个Latent Diffusion Models，该模型专门用于文图生成。

下图是Stable Diffusion生成图片的具体过程：

![img](\img\images\v2-33714d356fa27e628e1869c5aca1deae_r.jpg)

可以看到，对于输入的文字（图中的“An astronout riding a horse”）会经过一个CLIP模型转化为text embedding，然后和初始图像（初始化使用随机高斯噪声Gaussian Noise）一起输入去噪模块（也就是图中Text conditioned latent U-Net），最后输出 512×512 大小的图片。在文章（[绝密伏击：十分钟读懂Diffusion：图解Diffusion扩散模型](https://zhuanlan.zhihu.com/p/599887666)）中，我们已经知道了CLIP模型和U-Net模型的大致原理，这里面关键是**Text conditioned latent U-net**，翻译过来就是**文本条件隐U-net网络**，其实是**通过对U-Net引入多头Attention机制，使得输入文本和图像相关联**。

原理涉及的知识比较多，CLIP、U-Net、VAE、LORA、扩散、反向扩散，等等。[这里](https://zhuanlan.zhihu.com/p/600251419)有个文章可以研读一下。

用Stable Diffusion既可以画接近真实世界的模型，也可以画类似动画的二次元模型，还有人画两者之间的那种，反正不一而足。chilloutmix_NiPrunedFp32Fix大模型主要用于生成类似真实图片，anything、anime等大模型就是二次元的了。

#### Lora训练

AI画图少不了训练自己的Lora，可以把自己的照片训练成Lora，这样你想要去哪里得照片，让电脑给你生成就行了。Lora可以理解为是小模型，在大模型的基础上对照片进行微调。

Clone repo with submodules

```sh
git clone --recurse-submodules https://github.com/Akegarasu/lora-scripts
```

Required Dependencies

* Python 3.10.8 and Git

具体训练可以参照文档。

```shell
# run train
accelerate launch --num_cpu_threads_per_process=8 "./sd-scripts/train_network.py" `
  --enable_bucket `
  --pretrained_model_name_or_path=$pretrained_model `
  --train_data_dir=$train_data_dir `
  --output_dir="./output" `
  --logging_dir="./logs" `
  --resolution=$resolution `
  --network_module=$network_module `
  --max_train_epochs=$max_train_epoches `
  --learning_rate=$lr `
  --unet_lr=$unet_lr `
  --text_encoder_lr=$text_encoder_lr `
  --lr_scheduler=$lr_scheduler `
  --lr_warmup_steps=$lr_warmup_steps `
  --lr_scheduler_num_cycles=$lr_restart_cycles `
  --network_dim=$network_dim `
  --network_alpha=$network_alpha `
  --output_name=$output_name `
  --train_batch_size=$batch_size `
  --save_every_n_epochs=$save_every_n_epochs `
  --mixed_precision="no" `
  --save_precision="fp16" `
  --seed="1337" `
  --cache_latents `
  --clip_skip=$clip_skip `
  --prior_loss_weight=1 `
  --max_token_length=225 `
  --caption_extension=".txt" `
  --save_model_as=$save_model_as `
  --min_bucket_reso=$min_bucket_reso `
  --max_bucket_reso=$max_bucket_reso `
  --keep_tokens=$keep_tokens `
   $ext_args
Write-Output "Train finished"
Read-Host | Out-Null ;
```

这里要注意一个单精度、双精度、半精度的概念。基本上，这是显卡能力的表现，双精度就是64位的浮点数，单精度32位，半精度就是16位了。训练的时候要注意自己显卡能支持哪个精度。

## 爆火的Midjourney

这里也说说闭源但是爆火的Midjourney。其创始人是David Holz。目前Midjourney每月的收入大概超过**200万美元**，用户可以通过Discord平台的newbie频道使用。本质上Discord是一个社区，Midjourney通过在Discord上创建了自己的服务器，并创建了大量的频道，以及开发了自己的机器人，来向用户提供服务。

本来Midjourney可以免费试用的，但是因为来自中国薅羊毛的太多，所以我们现在没办法再免费试用了。无语。。。

Midjourney由David Holz创立于2021年8月，仅拥有11名全职员工，成立至今未融过资，却凭借着付费订阅的商业模式，实现年营收1亿美元。是个牛人。

### 总结

1. 以ChatGPT的出现为标志，人类彻底进入了智能时代
1. AI绘画的出现，已经让很多做原图的美工失业了
1. 不仅如此，好像很多模特、服装设计等工作，AI绘画也能做
1. AI蛰伏这么多年，终于到了爆发的时刻。未来人手一个“贾维斯”不是梦
1. 为啥颠覆式的创新又出现在美国？

---
layout:     post
title:      AI不得了之四：CPU+GPU终于跑起了Llama2的70B大模型
subtitle:   大模型逐步走进寻常百姓家
date:       2023-11-13
author:     就是我啦
header-img: img/post-bg-keybord.jpg
catalog: 	  true
tags:  
    - Llama2
    - AI
    - 人工智能
    - llama.cpp
    - ChatGLM3      
    - GGML    
    - GGUF    

---

# AI不得了之四：CPU+GPU终于跑起了Llama2的70B大模型

> 各种大模型如雨后春笋似的不断的冒出来，其中开源的也越来越多。这其中又有Llama2的影响比较大。
>
> 最近在熟悉VS下的.NET开发，不经意间看到了llama.cpp和其在.NET下的绑定llamaCsharp，遂尝试一番，没想到还不错。

```sh
在我的两台家用主机上，不仅Llama2的13B量化模型跑的很好，甚至于连70B模型都能在我的22G魔改2080Ti+64G内存下跑起来，属实惊喜。

```

## 引子

能有惊人表现的根源来自于llama.cpp。这是个富有传奇色彩的项目。一般来说，传统大模型体积巨大，非一般家用电脑显卡所能处理。问题包括：

1. LLMs 动辄数十上百亿甚至上千亿的参数，对运行机器的内存提出了很高的要求，毕竟只有将模型权重塞进 RAM，推理方可进行；
2. 模型加载至内存后，推理顺畅与否，又与 CPU、GPU 等计算单元密切相关，要知道很多大语言模型是在顶级专用 GPU 集群上加速训练的，换到个人电脑上，五秒蹦出一个词，也很难说用了起来。

在此挑战下，就有大神 Georgi Gerganov站了出来（据说[仅仅是一个晚上的 hacking](https://link.zhihu.com/?target=https%3A//github.com/ggerganov/llama.cpp/issues/33%23issuecomment-1465108022)），其**基于 Meta 释出的 LLaMA 模型（简易 Python 代码示例）手撸了纯 C/C++ 版本**，即llama.cpp，用于模型推理。所谓推理，即是给输入-跑模型-得输出的模型运行过程。

那么，纯 C/C++ 版本有何优势呢？

- 无需任何额外依赖，相比 Python 代码对 PyTorch 等库的要求，C/C++ 直接编译出可执行文件，跳过不同硬件的繁杂准备；
- 支持 Apple Silicon 芯片的 ARM NEON 加速，x86 平台则以 AVX2 替代；
- 具有 F16 和 F32 的混合精度；
- **支持 4-bit 量化；**
- **无需 GPU，可只用 CPU 运行；**也可两者结合运行。

那么模型是如何装进内存的呢？LLaMA.cpp 的答案是**量化**。



## 关于量化

深度神经网络模型在结构设计好之后，训练过程的核心目的是确定每个神经元的**权重参数**，通常是记为**浮点数**，精度有 **16、32、64** 位不一，基于 GPU 加速训练所得，量化就是通过将这些权重的精度降低，以降低硬件要求的过程。

举例而言，LLaMA 模型为 16 位浮点精度，其 7B 版本有 70 亿参数，该模型完整大小为 13 GB，则用户至少须有如此多的内存和磁盘，模型才能可用，更不用提 13B 版本 25 GB 的大小，令人望而却步。但通过量化，比如将精度降至 **4 位整数，**则 7B 和 13B 版本分别压至约 4 GB 和 **8 GB**，消费级硬件即可满足要求，大家便能在个人电脑上体验大模型了。

LLaMA.cpp 的量化实现基于作者的另外一个库—— **ggml**，使用 C/C++ 实现的机器学习模型中的 tensor。所谓 tensor，其实是神经网络模型中的核心数据结构，常见于 TensorFlow、PyTorch 等框架。改用 C/C++ 实现后，支持更广，效率更高，也为 LLaMA.cpp 的出现奠定了基础。由于核心在于 ggml 这个 tensor 库，在社区广为应用的情况下，大家也用 `ggml` 格式来称呼此类经过转换的模型，于是大哥 GG 便冠名定义了一种格式。后来，其又对ggml格式进行了改进，称为**gguf**。



## 操作步骤

### 1. 克隆llama.cpp项目到本地

大神项目地址，https://github.com/ggerganov/llama.cpp

### 2.下载Llama2大模型

大模型一般可以从huggingface下载，可因为某种“不知道“的原因，现在下载不了了。好在国内还有阿里的modelscope可以下载。大模型确实大，70B的有260G，包含两种格式，当格式的也有130G了。13B的有25G。

### 3. 转换模型为gguf格式f16

llama.cpp下有个转换工具，直接运行就好：

```shell
# obtain the original LLaMA model weights and place them in ./models
ls ./models
65B 30B 13B 7B tokenizer_checklist.chk tokenizer.model
  # [Optional] for models using BPE tokenizers
  ls ./models
  65B 30B 13B 7B vocab.json

# install Python dependencies
python3 -m pip install -r requirements.txt

# convert the 7B model to ggml FP16 format
python3 convert.py models/7B/

  # [Optional] for models using BPE tokenizers
  python convert.py models/7B/ --vocabtype bpe

# quantize the model to 4-bits (using q4_0 method)
./quantize ./models/7B/ggml-model-f16.gguf ./models/7B/ggml-model-q4_0.gguf q4_0

# update the gguf filetype to current if older version is unsupported by another application
./quantize ./models/7B/ggml-model-q4_0.gguf ./models/7B/ggml-model-q4_0-v2.gguf COPY


# run the inference
./main -m ./models/7B/ggml-model-q4_0.gguf -n 128
```

### 4. 量化转换

llama.cpp下的转换工具同样可以进行量化转换，上面的代码中已经展现。

量化包括多种格式，一般q4_0压缩体积最小。

量化转换还可以用另一个项目LLamaSharp来完成。这是项目是**The C#/.NET binding of [llama.cpp](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fggerganov%2Fllama.cpp).**，也就是llama.cpp在.NET和C#下的版本。当然llama.cpp还有支持其他语言的版本，包括java、python等。

![image-20231113160351087](\img\images\image-20231113160351087.png)

首先，要安装包：

```c#
LLamaSharp.Backend.Cpu  # cpu for windows, linux and mac (mac metal is also supported)
LLamaSharp.Backend.Cuda11  # cuda11 for windows and linux
LLamaSharp.Backend.Cuda12  # cuda12 for windows and linux
LLamaSharp.Backend.MacMetal  # special for using mac metal
```

然后就可以通过一段C#代码来实现量化转换：

```c#
string srcFilename = "<Your source path>";
string dstFilename = "<Your destination path>";
string ftype = "q4_0";
if(Quantizer.Quantize(srcFileName, dstFilename, ftype))
{
    Console.WriteLine("Quantization succeed!");
}
else
{
    Console.WriteLine("Quantization failed!");
}
```

### 5. 调用量化后的模型进行推理

LLamaSharp下可以直接调用量化后的模型进行推理，代码如下：

```c#
using LLama.Common;
using LLama;

string modelPath = "<Your model path>"; // change it to your own model path
var prompt = "Transcript of a dialog, where the User interacts with an Assistant named Bob. Bob is helpful, kind, honest, good at writing, and never fails to answer the User's requests immediately and with precision.\r\n\r\nUser: Hello, Bob.\r\nBob: Hello. How may I help you today?\r\nUser: Please tell me the largest city in Europe.\r\nBob: Sure. The largest city in Europe is Moscow, the capital of Russia.\r\nUser:"; // use the "chat-with-bob" prompt here.

// Load a model
var parameters = new ModelParams(modelPath)
{
    ContextSize = 1024,
    Seed = 1337,
    GpuLayerCount = 5
};
using var model = LLamaWeights.LoadFromFile(parameters);

// Initialize a chat session
using var context = model.CreateContext(parameters);
var ex = new InteractiveExecutor(context);
ChatSession session = new ChatSession(ex);

// show the prompt
Console.WriteLine();
Console.Write(prompt);

// run the inference in a loop to chat with LLM
while (prompt != "stop")
{
    foreach (var text in session.Chat(prompt, new InferenceParams() { Temperature = 0.6f, AntiPrompts = new List<string> { "User:" } }))
    {
        Console.Write(text);
    }
    prompt = Console.ReadLine();
}

// save the session
session.SaveSession("SavedSessionPath");
```



这其中，参数**GpuLayerCount**用于标识**有多少层的模型参数会被装载到GPU显存**中，其余层则会**装载到内存**中。显存中的模型层可以利用显存的更高速度和显卡的更强大CUDA运算能力。因而，GpuLayerCount的设置一般取决于你的显存有多大。比如，在我的2080Ti22G显存下，13B大模型的8bit量化可以全部装载到显存中，运行起来简直飞快。

我另一台电脑用了8G显存的P104-100，经过测试，对于Llama2的13B模型，其参数设置如下：

```c#
GpuLayerCount = 50   // for q4_0
//GpuLayerCount = 40   // for q4_1
//GpuLayerCount = 31   // for q5_1
//GpuLayerCount = 22  // for q8_0
```



# 结论

llama.cpp还是很好用的，缓解了我们GPU显存不足的尴尬，可以显存内存一起用，GPU+CPU一起跑，让我们在自己的电脑上也能体验一般大模型的瘾。

不过，其并不是什么模型都支持，感觉好像主要是llama系的：

![image-20231113160627965](\img\images\image-20231113160627965.png)

好消息是，里面居然有百川。。。这是不是说明百川是从。。。？

国内的ChatGLM和阿里开源的千问都不在其中。

不过，据说有chatglm.cpp，干了类似的事情，也可以搞一下的。


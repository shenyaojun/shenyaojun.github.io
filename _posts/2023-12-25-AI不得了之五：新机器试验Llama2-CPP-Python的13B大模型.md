---
layout:     post
title:      AI不得了之五：新机器试验Llama2-CPP-Python的13B大模型
subtitle:   矿渣也能跑AI
date:       2023-12-25
author:     就是我啦
header-img: img/post-bg-purplepalace.png
catalog: 	  true
tags:  
    - Llama2
    - AI
    - 人工智能
    - llama.cpp
    - llama-cpp-python      
    - ChatGPT
    - CUDA

---

# AI不得了之五：新机器试验Llama2-CPP-Python的13B大模型

> 无聊之余捡垃圾弄了台新机器，4代I3，16G内存，感觉跑的还挺好。更无聊的弄了个矿渣显卡P106-90，3G的显存，突然想知道这玩意儿能不能跑AI？

```sh
P106系列是知名的矿渣，P106-90更是低端，居然是PCIE1.1X4的，不过本来我也不玩游戏，就当测测AI的极限性能吧。

```

## 引子

以前用量化后的Llama2大模型利用CPU+GPU跑了AI，心里想P106当然也可以，无非是显存小点，那就多用内存少用显存就是了，大不了慢一点。

赶紧用新机器证实一下。

## 用C#尝试

当然新机器得先安装P106的魔改驱动、CUDA等，再下载安装Visual Studio。然后再把以前机器上用的C#程序直接拿过来，然后安装C#的包，发现下面的包其实有点问题的：

```
LLamaSharp.Backend.Cpu  # cpu for windows, linux and mac (mac metal is also supported)
LLamaSharp.Backend.Cuda11  # cuda11 for windows and linux
LLamaSharp.Backend.Cuda12  # cuda12 for windows and linux
LLamaSharp.Backend.MacMetal  # special for using mac metal
```

就是如果想利用GPU的话，就不能安装LLamaSharp.Backend.Cpu。有这个包，就会自动直接调CPU，不利用CUDA。删除这个包后，才会自动利用CUDA。

例外，就是新版（0.8.X）的LLamaSharp.Backend.Cuda11明显比老版本0.7.X快了。我跑的llama2 13B的q4量化版本，在这台矿渣+老机器上居然还跑的挺快。

```c#
          var parameters = new ModelParams(modelPath)
            {
                ContextSize = 1024,
                Seed = 1337,
                GpuLayerCount = 15
            };
            using var model = LLamaWeights.LoadFromFile(parameters);
            using var context = model.CreateContext(parameters);
            var executor = new InteractiveExecutor(context);

            var session = new ChatSession(executor).WithOutputTransform(new LLamaTransforms.KeywordTextOutputStreamTransform(new string[] { "User:", "Bob:" }, redundancyLength: 8));

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("The chat session has started. The role names won't be printed.");
            Console.ForegroundColor = ConsoleColor.White;

            // show the prompt
            Console.Write(prompt);
            while (true)
            {
                await foreach (var text in session.ChatAsync(prompt, new InferenceParams() { Temperature = 0.6f, AntiPrompts = new List<string> { "User:" } }))
                {
                    Console.Write(text);
                }

                Console.ForegroundColor = ConsoleColor.Green;
                prompt = Console.ReadLine();
                Console.ForegroundColor = ConsoleColor.White;
            }
```

模型一共41层，15层放GPU的话，好像3G内存不够用，10层放GPU 3G内存又用不满，估计12层差不多。

## 尝试Python版Llama2-CPP-Python

### 1. 安装

python版正常的安装命令很简单：

```shell
pip install llama-cpp-python
```

但是这个安装只能安装CPU版本，需要GPU的话要用这个：

```sh
CMAKE_ARGS="-DLLAMA_CUBLAS=on" pip install llama-cpp-python
```

但这个命令刚开始运行的时候是报错的：No CUDA toolset found.

一通百度之后，好像用这个才解决：

https://blog.csdn.net/weixin_46363611/article/details/126177727

https://blog.csdn.net/qq_33642342/article/details/126956116

以cuda11.7为例，将

C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.7\extras\visual_studio_integration\MSBuildExtensions
下的所有文件
拷贝到
C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\BuildCustomizations
下
具体目录根据自己的版本调整。

### 2. Llama2 API调用

Python的安装我直接用的Anaconda安装包，很方便。

```python
>>> from llama_cpp import Llama
>>> llm = Llama(model_path="./models/7B/llama-model.gguf")
>>> output = llm(
      "Q: Name the planets in the solar system? A: ", # Prompt
      max_tokens=32, # Generate up to 32 tokens
      last_n_tokens_size=1024,
            seed=1337,
            n_gpu_layers=10
      stop=["Q:", "\n"], # Stop generating just before the model would generate a new question
      echo=True # Echo the prompt back in the output
) # Generate a completion, can also call create_completion
>>> print(output)
{
  "id": "cmpl-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "object": "text_completion",
  "created": 1679561337,
  "model": "./models/7B/llama-model.gguf",
  "choices": [
    {
      "text": "Q: Name the planets in the solar system? A: Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune and Pluto.",
      "index": 0,
      "logprobs": None,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 14,
    "completion_tokens": 28,
    "total_tokens": 42
  }
}
```

测试很容易就成功了，这里面要注意的是n_gpu_layers这个参数。它指定了放入到GPU的层数。

### 3. functionary模式测试

这个模式挺有意思，可以对你的一段文字进行语义分析，抽取你想要的信息并以指定的格式给你：

```python
from llama_cpp import Llama
llm = Llama(model_path="F:/Download/bilibili_llama2/Llama2-Chinese-13b-Chat-ms/ggml-model-q4_0.gguf",
            chat_format="functionary",
            n_ctx=1024, # context window size
            seed=1337,
            n_gpu_layers=10)
            

output = llm.create_chat_completion(
      messages = [
        {
          "role": "system",
          "content": "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. The assistant calls functions with appropriate input when necessary"
          
        },
        {
          "role": "user",
          "content": "Extract Jason is 25 years old"
        }
      ],
      tools=[{
        "type": "function",
        "function": {
          "name": "UserDetail",
          "parameters": {
            "type": "object",
            "title": "UserDetail",
            "properties": {
              "name": {
                "title": "Name",
                "type": "string"
              },
              "age": {
                "title": "Age",
                "type": "integer"
              }
            },
            "required": [ "name", "age" ]
          }
        }
      }]
)
print(output)
```

最后的结果里会按照[ "name", "age" ]的JSON串返回给你。

### 4. WebServer 模式

最后一种有意思的模式是WebServer模式。运行这个模式需要先安装一下

```sh
 CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python[server]
```

然后用以下命令就可以启动一个大模型的服务

```sh
python -m llama_cpp.server --model F:/Download/bilibili_llama2/Llama2-Chinese-13b-Chat-ms/ggml-model-q4_0.gguf --n_gpu_layers 10
```

模型启动后输出如下：

```sh
ggml_init_cublas: GGML_CUDA_FORCE_MMQ:   no
ggml_init_cublas: CUDA_USE_TENSOR_CORES: yes
ggml_init_cublas: found 1 CUDA devices:
  Device 0: NVIDIA P106-090, compute capability 6.1
llama_model_loader: loaded meta data with 19 key-value pairs and 363 tensors from F:/Download/bilibili_llama2/Llama2-Chinese-13b-Chat-ms/ggml-model-q4_0.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = llama
llama_model_loader: - kv   1:                               general.name str              = F:\Download\bilibili_llama2
llama_model_loader: - kv   2:                       llama.context_length u32              = 2048
llama_model_loader: - kv   3:                     llama.embedding_length u32              = 5120
llama_model_loader: - kv   4:                          llama.block_count u32              = 40
llama_model_loader: - kv   5:                  llama.feed_forward_length u32              = 13824
llama_model_loader: - kv   6:                 llama.rope.dimension_count u32              = 128
llama_model_loader: - kv   7:                 llama.attention.head_count u32              = 40
llama_model_loader: - kv   8:              llama.attention.head_count_kv u32              = 40
llama_model_loader: - kv   9:     llama.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  10:                          general.file_type u32              = 2
llama_model_loader: - kv  11:                       tokenizer.ggml.model str              = llama
llama_model_loader: - kv  12:                      tokenizer.ggml.tokens arr[str,32000]   = ["<unk>", "<s>", "</s>", "<0x00>", "<...
llama_model_loader: - kv  13:                      tokenizer.ggml.scores arr[f32,32000]   = [0.000000, 0.000000, 0.000000, 0.0000...
llama_model_loader: - kv  14:                  tokenizer.ggml.token_type arr[i32,32000]   = [2, 3, 3, 6, 6, 6, 6, 6, 6, 6, 6, 6, ...
llama_model_loader: - kv  15:                tokenizer.ggml.bos_token_id u32              = 1
llama_model_loader: - kv  16:                tokenizer.ggml.eos_token_id u32              = 2
llama_model_loader: - kv  17:            tokenizer.ggml.padding_token_id u32              = 0
llama_model_loader: - kv  18:               general.quantization_version u32              = 2
llama_model_loader: - type  f32:   81 tensors
llama_model_loader: - type  f16:    1 tensors
llama_model_loader: - type q4_0:  281 tensors
llm_load_vocab: special tokens definition check successful ( 259/32000 ).
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = llama
llm_load_print_meta: vocab type       = SPM
llm_load_print_meta: n_vocab          = 32000
llm_load_print_meta: n_merges         = 0
llm_load_print_meta: n_ctx_train      = 2048
llm_load_print_meta: n_embd           = 5120
llm_load_print_meta: n_head           = 40
llm_load_print_meta: n_head_kv        = 40
llm_load_print_meta: n_layer          = 40
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: n_ff             = 13824
llm_load_print_meta: n_expert         = 0
llm_load_print_meta: n_expert_used    = 0
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 10000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_yarn_orig_ctx  = 2048
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: model type       = 13B
llm_load_print_meta: model ftype      = Q4_0
llm_load_print_meta: model params     = 13.02 B
llm_load_print_meta: model size       = 7.04 GiB (4.65 BPW)
llm_load_print_meta: general.name     = F:\Download\bilibili_llama2
llm_load_print_meta: BOS token        = 1 '<s>'
llm_load_print_meta: EOS token        = 2 '</s>'
llm_load_print_meta: UNK token        = 0 '<unk>'
llm_load_print_meta: PAD token        = 0 '<unk>'
llm_load_print_meta: LF token         = 13 '<0x0A>'
llm_load_tensors: ggml ctx size       =    0.14 MiB
llm_load_tensors: using CUDA for GPU acceleration
llm_load_tensors: system memory used  = 5506.41 MiB
llm_load_tensors: VRAM used           = 1701.95 MiB
llm_load_tensors: offloading 10 repeating layers to GPU
llm_load_tensors: offloaded 10/41 layers to GPU
................................................................................................
llama_new_context_with_model: n_ctx      = 2048
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 1
llama_new_context_with_model: KV self size  = 1600.00 MiB, K (f16):  800.00 MiB, V (f16):  800.00 MiB
llama_build_graph: non-view tensors processed: 844/844
llama_new_context_with_model: compute buffer total size = 197.19 MiB
llama_new_context_with_model: VRAM scratch buffer: 194.00 MiB
llama_new_context_with_model: total VRAM used: 1895.96 MiB (model: 1701.95 MiB, context: 194.00 MiB)
AVX = 1 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | FMA = 1 | NEON = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | 
= 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 0 | VSX = 0 |
INFO:     Started server process [6968]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8000 (Press CTRL+C to quit)
INFO:     ::1:2478 - "GET / HTTP/1.1" 404 Not Found
INFO:     ::1:2479 - "GET /chat HTTP/1.1" 404 Not Found
INFO:     ::1:2484 - "GET / HTTP/1.1" 404 Not Found
INFO:     ::1:2484 - "GET / HTTP/1.1" 404 Not Found

llama_print_timings:        load time =   12429.35 ms
llama_print_timings:      sample time =       1.92 ms /     8 runs   (    0.24 ms per token,  4171.01 tokens per second)
llama_print_timings: prompt eval time =   12429.22 ms /    57 tokens (  218.06 ms per token,     4.59 tokens per second)
llama_print_timings:        eval time =    3304.00 ms /     7 runs   (  472.00 ms per token,     2.12 tokens per second)
llama_print_timings:       total time =   15865.07 ms
INFO:     ::1:2664 - "POST /v1/chat/completions HTTP/1.1" 200 OK
INFO:     ::1:2679 - "GET / HTTP/1.1" 404 Not Found
INFO:     ::1:2679 - "GET / HTTP/1.1" 404 Not Found
Llama.generate: prefix-match hit

llama_print_timings:        load time =   12429.35 ms
llama_print_timings:      sample time =      57.96 ms /   276 runs   (    0.21 ms per token,  4761.58 tokens per second)
llama_print_timings: prompt eval time =   12584.79 ms /    29 tokens (  433.96 ms per token,     2.30 tokens per second)
llama_print_timings:        eval time =  144603.47 ms /   275 runs   (  525.83 ms per token,     1.90 tokens per second)
llama_print_timings:       total time =  158017.88 ms
INFO:     ::1:2704 - "POST /v1/chat/completions HTTP/1.1" 200 OK

```

官方文档说Navigate to [http://localhost:8000/docs](http://localhost:8000/docs) to see the OpenAPI documentation。这里有API的说明，可以写程序调用。

![image-20231225112535434](/img/images/image-20231225112535434.png)

下面是一个调用的示例：

```python
import requests
  
url = 'http://localhost:8000/v1/chat/completions'
headers = {
	'accept': 'application/json',
	'Content-Type': 'application/json'
}
data = {
	'messages': [
		{
		'content': 'You are a helpful assistant.',
		'role': 'system'
		},
		{
		'content': 'please write a story about a cat and a dog.',
		'role': 'user'
		}
	]
}
  
response = requests.post(url, headers=headers, json=data)
print(response.json())
print(response.json()['choices'][0]['message']['content'])
```

运行，等一下，大概两分钟，输出：

```sh
{'id': 'chatcmpl-4c2de099-2344-42f4-9d4e-e95095d3860f', 'object': 'chat.completion', 'created': 1703471717, 'model': 'F:/Download/bilibili_llama2/Llama2-Chinese-13b-Chat-ms/ggml-model-q4_0.gguf', 'choices': [{'index': 0, 'message': {'content': 'Once upon a time, there was a mischievous cat named Whiskers who lived in a cozy little house with his best friend, a loyal and lovable dog named Duke. Whiskers loved to explore the world outside of their home and would often go on long adventures, but he always returned to find comfort and companionship with Duke.\nOne day, while out exploring, Whiskers stumbled upon an adorable little kitten who had gotten lost from her mother. Feeling sympathetic towards the tiny creature, Whiskers decided to take the kitten home and introduce her to Duke. However, Duke was not pleased with the new addition to their family and chased the kitten around the house, causing Whiskers great distress.\nFeeling guilty for neglecting his friendship with Duke, Whiskers decided to make amends by spending more time with his loyal companion. From then on, Whiskers made sure to always include Duke in his adventures and even allowed him to take charge of the house when Whiskers was away exploring.\nThe three friends lived happily ever after, with Whiskers learning that having a loyal companion like Duke by your side can make any adventure more enjoyable, and that sometimes making mistakes can lead to unexpected and wonderful new experiences.|', 'role': 'assistant'}, 'finish_reason': 'stop'}], 'usage': {'prompt_tokens': 61, 'completion_tokens': 275, 'total_tokens': 336}}
Once upon a time, there was a mischievous cat named Whiskers who lived in a cozy little house with his best friend, a loyal and lovable dog 
named Duke. Whiskers loved to explore the world outside of their home and would often go on long adventures, but he always returned to find 
comfort and companionship with Duke.
One day, while out exploring, Whiskers stumbled upon an adorable little kitten who had gotten lost from her mother. Feeling sympathetic towards the tiny creature, Whiskers decided to take the kitten home and introduce her to Duke. However, Duke was not pleased with the new addition to their family and chased the kitten around the house, causing Whiskers great distress.
Feeling guilty for neglecting his friendship with Duke, Whiskers decided to make amends by spending more time with his loyal companion. From then on, Whiskers made sure to always include Duke in his adventures and even allowed him to take charge of the house when Whiskers was away exploring.
The three friends lived happily ever after, with Whiskers learning that having a loyal companion like Duke by your side can make any adventure more enjoyable, and that sometimes making mistakes can lead to unexpected and wonderful new experiences.|
```

小故事还是不错的：）



# 结论

* 矿渣跑AI没问题
* webserver方式挺好，可以开放web API给其他程序调用
* functionary模式也有意义，一些商业环境下可以辅助抽取有用信息


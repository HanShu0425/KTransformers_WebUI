# KTransformers_WebUI
低成本2*2080Ti部署满血DeepSeek-R1模型 : 技术栈： KTransformers + DeepSeek（gguf版） + OpenWebUI版

环境： 2*2080Ti

通过在服务器上部署KTransformers后 加载DeepSeek的参数文件

1. 发现2080无法使用KTrans自带的Marlin算子
2. 当部署KTrans自带的UI后出现 Request Error问题后，选择用OpenWebUI实现

### 部署KTransformer

[Installation Guide - Ktransformers](https://kvcache-ai.github.io/ktransformers/en/install.html)

按照官方部署 完成Preparation、Installation以及测试的 Local Chat

### 下载 DeekSeek-R1参数

##### 注意： 下载DeepSeek-R1时 不能使用原版的.safetensors文件，而是他们提供的.gguf格式文件

[huggingface.co](https://huggingface.co/unsloth/DeepSeek-R1-GGUF/tree/main/DeepSeek-R1-Q4_K_M)

# 部署问题：

## 1.如果使用.git下载时 

```
git clone https://github.com/kvcache-ai/ktransformers.git

cd ktransformers 

git submodule init 

git submodule update
```

不会出现任何问题

##### 但是如果部署的是 zip下载的方法，会出现版本不一致的情况

```
fatal error:l.h: No such file or directory25|
#include "llama.cpp/ggml-impl.h"
llama.cpp/ggml-imp
compilation terminated.
```

导致 第三方插件 `llama.cpp` 和 `pybind11` 无法正确识别

#### 解决办法：

尝试用gitee下载KTrans模块

`git clone https://gitee.com/your/path/ktransformers.git  `

两个第三方软件也都更换成gitee的同源软件 in `ktransformers/.gitmodules`

```
[submodule "third_party/llama.cpp"]
	path = third_party/llama.cpp
	url = https://your/path/llama.cpp.git
[submodule "third_party/pybind11"]
	path = third_party/pybind11
	url = https://your/path/pybind11.git
```

## 2. CUDA_LAUNCH_BLOCKING=1

正常做`Local_Chat`

```
python -m ktransformers.local_chat --model_path ./DeepSeek-R1  --gguf_path ./deepseek_ggufs
```

```
RuntimeError: CUDA error: invalid device function
CUDA kernel errors might be asynchronously reported at some other API call, so the stacktrace below might be incorrect.
For debugging consider passing CUDA_LAUNCH_BLOCKING=1
Compile with `TORCH_USE_CUDA_DSA` to enable device-side assertions.
```

#### 原因：Marlins算子对GPU有要求

[再次提一下，能否支持2080ti · Issue #150 · kvcache-ai/ktransformers](https://github.com/kvcache-ai/ktransformers/issues/150)

##### 解决方法：

##### 注：找到自己配置的 DeepSeek-V2-Chat-multi-gpu.yaml文件进行更新

```
# generate_op: "KLinearMarlin" 
generate_op: "KLinearTorch" # change to KlinearTorch
```



## 3.KTrans前端UI： Request Error

参考链接：

[WebUI API请求URL问题 · Issue #515 · kvcache-ai/ktransformers](https://github.com/kvcache-ai/ktransformers/issues/515)

##### 输入网址：[KTransformers](http://localhost:10002/web/index.html#/chat)本地LocalHost可正常运行

##### 但[KTransformers](http://ip:10002/web/index.html#/chat)使用ip会显示UI界面，但运行时出现 Request Error

#### 原因：走了`localhost`的地址，这个地方应该换成`/`而非`localhost:port/`

##### 解决办法1：增加SSH端口 ssh 映射到本地，通过 localhost 访问

##### 但是该方法治标不治本，所以团队通过使用OpenAI WebUI绕过KTrans的UI

##### 解决方法2：Open-WebUI 使用 KTransformer

[通过 Open-WebUI 使用 KTransformer · Issue #578 · kvcache-ai/ktransformers](https://github.com/kvcache-ai/ktransformers/issues/578)

```
open-webui serve
```

[Open WebUI](http://ip:8080/) 
[Open WebUI](http://ip:8080/) 

在`ktransformers/ktransformers/website/public/index.html`

修改`<script src="./config_remote.js">` 成你自己的config文件

```
window.configWeb = {
    apiUrl: '/v1',
    port: 8080,
  };
```

后只需要配置编辑链接

URL： http://ip:10002/v1 密钥与ID随便设置即可！



有问题私信+V： Shu19857404990

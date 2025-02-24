## Low-cost 2*2080Ti deployment of full-blooded DeepSeek-R1 model
### Technology Stack: KTransformers + DeepSeek (gguf version) + OpenWebUI version
Environment: 2*2080Ti

After deploying KTransformers on the server and loading DeepSeek's parameter file, I found that the 2080 could not use KTransformers with the DeepSeek-R1 model.

I found that 2080 cannot use KTrans' Marlin operator.

After deploying KTrans UI with Request Error problem, I chose to implement it with OpenWebUI.

### Deploying KTransformer
Installation Guide - Ktransformers

Follow the official deployment Complete the Preparation, Installation and testing of Local Chat.

### Download DeekSeek-R1 parameters
#### Note: When downloading DeepSeek-R1, you can't use the original .safetensors file, but the .gguf file they provide.
[huggingface.co](https://huggingface.co/unsloth/DeepSeek-R1-GGUF/tree/main/DeepSeek-R1-Q4_K_M)

## Deployment issues:

## 1. If using .git when downloading 

```
git clone https://github.com/kvcache-ai/ktransformers.git

cd ktransformers 

git submodule start 

Updating a submodule
```

There is no problem

##### But if the zip download method is deployed, there will be version inconsistencies

```
fatal error:l.h: No such file or directory25|
#include “llama.cpp/ggml-impl.h” #include “llama.cpp/ggml-impl.h”.
llama.cpp/ggml-imp
Compilation terminated.
```

Causes third-party plugins `llama.cpp` and `pybind11` to not be recognized correctly

#### Solution:

Try to download the KTrans module with gitee

git clone https://gitee.com/nemo1982/ktransformers.git ` Both third-party programs are also replaced with git.

Both third-party programs are also replaced with the same source from gitee in `ktransformers/.gitmodules

```
[submodule “third_party/llama.cpp”)
	path = third_party/llama.cpp
	url = https://gitee.com/fastdgiot/llama.cpp.git
[submodule “third_party/pybind11”] path = third_party/pybind11.cpp
	path = third_party/pybind11
	url = https://gitee.com/uniqueinfo_395910063/pybind11.git
```

## 2. cuda_launch_blocking=1

Do `Local_Chat normally`

```
python -m ktransformers.local_chat --model_path . /DeepSeek-R1 --gguf_path . /deepseek_ggufs
```

```
RuntimeError: CUDA error: invalid device function
CUDA kernel errors may be reported asynchronously in other API calls, so the stack trace below may not be correct.
For debugging purposes, consider passing CUDA_LAUNCH_BLOCKING=1
Compile with `TORCH_USE_CUDA_DSA` to enable device-side assertions.
```

#### Reason: Marlins algorithm has GPU requirements

Translated with www.DeepL.com/Translator (free version)

[Again, mention if 2080ti can be supported - Issue #150 - kvcache-ai/ktransformers](https://github.com/kvcache-ai/ktransformers/issues/150)

Solution:
Note: Find your own configured DeepSeek-V2-Chat-multi-gpu.yaml file and update it
# generate_op: "KLinearMarlin"
generate_op: "KLinearTorch" # change to KlinearTorch
3.KTrans frontend UI: Request Error
Reference Link:

WebUI API request URL issue - Issue #515 - kvcache-ai/ktransformers

Input URL: KTransformers works fine with localhost.
However, KTransformers displays the UI when using ip, but runs with a Request Error.
Reason: localhost address is used, this place should be changed to / instead of localhost:port/.
Solution 1: Increase the SSH port ssh mapping to local, through the localhost access
But this method treats the symptoms but not the root cause, so the team bypassed the KTrans UI by using the OpenAI WebUI.
Solution 2: Open-WebUI using KTransformer
Using KTransformer via Open-WebUI - Issue #578 - kvcache-ai/ktransformers

open-webui serve
Open WebUI

In ktransformers/ktransformers/website/public/index.html

Modify <script src="./config_remote.js"> into your own config file

window.configWeb = {
    window.configWeb = { apiUrl: '/v1',
    apiUrl: '/v1', port: 8080,
  };
After that you just need to configure the edit link

URL: http://ip:10002/v1 The key and ID are set at random!

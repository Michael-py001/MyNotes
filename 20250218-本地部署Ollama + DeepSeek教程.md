# 20250218-本地部署Ollama + DeepSeek教程

## Ollama的下载&安装

从官方网站下载并安装：

> https://ollama.com/download

![image-20250218170253965](https://s2.loli.net/2025/02/18/qQtFYBwlnIsLjzP.png)

默认情况下模型会下载到C盘，而且模型一般都很大，最好改为存到其他盘保存。这里需要设置一个环境变量：

变量名为：OLLAMA_MODELS

变量值为：D:\LLM\OllamaLLM\models (自己随便建一个目录即可)

![image-20250218173627227](https://s2.loli.net/2025/02/18/Sp4q1MUXvYfA9xR.png)

## DeepSeek的部署

从ollama官网中，点击顶部导航Models，列表中选择deepseek-r1。

![image-20250218172924070](https://s2.loli.net/2025/02/18/Ph3q9xlSzCsHYEm.png)

一般性能的电脑选择7b模型即可

![image-20250218174252597](https://s2.loli.net/2025/02/18/VIJtEwx8OWAFMDh.png)

复制右上角的命令，在命令行中粘贴输入：`ollama run deepseek-r1:7b`，就会自动下载模型。

等下载完成后，再次执行该命令，就会进入到了大模型的对话中。

![image-20250218174816329](https://s2.loli.net/2025/02/18/8VuqO9eBSJNzshD.png)

## 从魔塔社区下载模型

ollama官方的下载方式虽然很方便，但是速度很慢，这里推荐从[魔塔社区](https://modelscope.cn/)下载，上面有丰富的各种模型。

比如选择这个模型：

> https://modelscope.cn/models/unsloth/DeepSeek-R1-Distill-Qwen-7B-GGUF/files

![image-20250218180238421](https://s2.loli.net/2025/02/18/AgXnQlyGTu154pv.png)

首先需要安装 modelscope

```bash
pip install modelscope
```

使用命令下载单个文件到指定本地文件夹

```
modelscope download --model unsloth/DeepSeek-R1-Distill-Qwen-7B-GGUF DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf --local_dir ./dir(你的下载目录)
```

下载完成后，你还需要在下载模型目录下创建一个Modelfile，输入以下：

```
FROM DeepSeek-R1-Distill-Qwen-7B-Q4_K_M.gguf
```

下一步就可以用ollama创建这个模型了，在该目录下打开cmd，输入：

```bash
ollama create DeepSeek-R1-Distill-Qwen-7B -f ./Modelfile
```

执行成功后，使用命令列出已安装的模型：

```
ollama l
```

![image-20250218181826360](https://s2.loli.net/2025/02/18/LScWUBkEthaKGfw.png)


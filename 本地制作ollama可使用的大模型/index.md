# 本地制作Ollama可使用的大模型(PyTorch & Safetensors)


# 本地制作Ollama可使用的大模型(PyTorch & Safetensors)

1. 下载代码

   ```bash
   git clone https://github.com/ollama/ollama.git
   cd ollama
   # 下载llama.cpp 子模块
   git submodule init
   git submodule update llm/llama.cpp
   
   # 安装python依赖项
   python3 -m venv llm/llama.cpp/.venv
   source llm/llama.cpp/.venv/bin/activate
   pip install -r llm/llama.cpp/requirements.txt
   
   # 构建量化工具
   make -C llm/llama.cpp quantize
   ```

2. 下载模型

   以https://huggingface.co/FlagAlpha/Llama2-Chinese-7b-Chat举例

   ```bash
   git clone https://huggingface.co/FlagAlpha/Llama2-Chinese-7b-Chat model
   ```

3. 转换模型

   ```bash
   python llm/llama.cpp/convert.py ./model --outtype f16 --outfile converted.bin
   # 量化模型
   llm/llama.cpp/quantize converted.bin quantized.bin q4_0
   ```

4. 编写`Modelfile`

   可参考对应原始模型

   例如

   ```bash
   ollama show --modelfile llama2
   ```

   即可看到对应模型的`Modelfile`

   修改FROM，其他不变即可

   ```
   FROM quantized.bin
   TEMPLATE """[INST] <<SYS>>{{ .System }}<</SYS>>
   
   {{ .Prompt }} [/INST]
   """
   PARAMETER stop "[INST]"
   PARAMETER stop "[/INST]"
   PARAMETER stop "<<SYS>>"
   PARAMETER stop "<</SYS>>"
   ```

5. 运行模型

   ```bash
   # 创建
   ollama create llama2-chinese -f Modelfile
   # 运行
   ollama run llama2-chinese
   ```

   



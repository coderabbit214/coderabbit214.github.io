# llama模型本地Running


# llama模型本地Running

## 代码准备

1. 下载代码

   ```bash
   git@github.com:coderabbit214/llama.cpp.git
   ```

2. 编译代码

   ```bash
   cd llama.cpp
   make
   ```

## Python环境准备

> 本机环境Python 3.10
>
> 可以使用
>
> ```bash
> pipenv shell --python 3.10
> ```
>
> 创建一个环境

1. 安装依赖

   ```bash
   pip3 install torch numpy sentencepiece
   ```

2. 把模型文件放到当前目录下

   `./models/7B/consolidated.00.pth` 文件大小应该是13GB

3. 将模型转化为"ggml FP16 format"

   ```bash
   python3 convert-pth-to-ggml.py models/7B/ 1
   ```

   生成文件：` ./models/7B/ggml-model-f16.bin` 大小：13GB

4. 将模型转化为4-bits

   > 注意 ：如果是多个模型，例如13B，则需要给每个模型运行命令
   >
   > ```bash
   > ./quantize ./models/13B/ggml-model-f16.bin   ./models/13B/ggml-model-q4_0.bin 2
   > ./quantize ./models/13B/ggml-model-f16.bin.1 ./models/13B/ggml-model-q4_0.bin.1 2
   > ```

   ```bash
   ./quantize ./models/7B/ggml-model-f16.bin ./models/7B/ggml-model-q4_0.bin 2
   ```

   生成文件：` ./models/7B/ggml-model-q4_0.bin` 大小：3.9GB

## Running

```bash
./main -m ./models/7B/ggml-model-q4_0.bin \
  -t 8 \
  -n 128 \
  -p 'The first man on the moon was '
```

### 参数说明

- -t:线程数
- -n:令牌数
- -p:提示

帮助：

```bash
./main -h
```

```
usage: ./main [options]

options:
  -h, --help            show this help message and exit
  -i, --interactive     run in interactive mode
  --interactive-start   run in interactive mode and poll user input at startup
  -r PROMPT, --reverse-prompt PROMPT
                        in interactive mode, poll user input upon seeing PROMPT
  --color               colorise output to distinguish prompt and user input from generations
  -s SEED, --seed SEED  RNG seed (default: -1)
  -t N, --threads N     number of threads to use during computation (default: 4)
  -p PROMPT, --prompt PROMPT
                        prompt to start generation with (default: random)
  -f FNAME, --file FNAME
                        prompt file to start generation.
  -n N, --n_predict N   number of tokens to predict (default: 128)
  --top_k N             top-k sampling (default: 40)
  --top_p N             top-p sampling (default: 0.9)
  --repeat_last_n N     last n tokens to consider for penalize (default: 64)
  --repeat_penalty N    penalize repeat sequence of tokens (default: 1.3)
  --temp N              temperature (default: 0.8)
  -b N, --batch_size N  batch size for prompt processing (default: 8)
  -m FNAME, --model FNAME
                        model path (default: models/llama-7B/ggml-model.bin)
```



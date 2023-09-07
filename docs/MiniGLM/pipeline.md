# 环境部署

* 安装transformers, gradio, torch等依赖包

  ```
  pip install transformers gradio torch
  ```

  在提供的服务器环境中，默认已经配置好所有依赖的包。

# MiniGLM 文件目录介绍

```bash
MiniGLM
├── README.md 
├── config
│   └── train_config.py # 模型参数文件
├── configurator.py
├── data
│   ├── fetch_data.py # 获取数据
│   ├── prepare.py # 准备预训练数据 <TODO>
│   └── prepare_sft.py # 准备微调数据 <TODO>
├── data_utils.py # 构造每一个训练步骤中要输入模型的数据 <TODO>
├── evaluations.py # 实现模型评估相关指标，辅助观察模型效果 <TODO>
├── model.py # 底层模型实现
├── sample.py # 使用训练好的模型进行推理，
├── sample_gradio.py # 使用 gradio 来进行和模型交互的可视化 <TODO> 也可以选用非 gradio 的其他实现
├── train.py # 模型训练实现
└── visualize.py # 模型训练 loss 曲线/评估指标等可视化 <TODO>
```

# 模型训练

模型训练包括`预训练`和`微调`两个阶段。

## 预训练模型（pretraing）

### 模型基本信息

本实验不涉及任何transformer模型结构上的实现及修改。如果你想要了解更多关于transformer或GPT模型结构的知识，可以阅读论文：
* transformers: https://arxiv.org/pdf/1706.03762.pdf
* GPT2: https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf

模型信息：
* 模型包含 45M 参数量
* 模型共包含 6 层transfomer，隐藏层包含 512 个单元。

### 预训练任务

MiniGLM采用GPT，也即自回归式的预训练及生成范式。
* 在预训练阶段：
  * 给定一段长为$N$的文本token序列。MiniGLM需要预测每一个位置的对应输出。对于位置$k$的token预测基于已给定的前$k-1$个token。
  * 对于模型输出的预测使用交叉熵（Cross Entropy Loss）构建损失函数来进行训练。
  * 对于每条预训练数据的采集，我们可以从原始数据中随机采样一个起始位置$i$，然后取$[i, i+N]$这一段序列作为本条预训练数据。
    ```python
    ix = torch.randint(len(data) - block_size, (batch_size,))
    x = torch.stack([torch.from_numpy((data[i:i+block_size]).astype(np.int64)) for i in ix])
    y = torch.stack([torch.from_numpy((data[i+1:i+1+block_size]).astype(np.int64)) for i in ix])
    ```

## 有监督微调模型（Supervised finetuning）



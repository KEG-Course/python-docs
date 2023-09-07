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

* 模型大小：包含 54M 参数，大约
  * 模型共包含 6 层transfomer，隐藏层包含 512 个单元。

## 有监督微调模型（Supervised finetuning）

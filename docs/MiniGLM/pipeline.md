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
├── data_utils.py # 
├── evaluations.py 
├── model.py
├── sample.py
├── sample_gradio.py
├── train.py
└── visualize.py
```



# 模型训练

模型训练包括`预训练`和`微调`两个阶段。

## 预训练模型（pretraing）

* 模型大小：包含 54M 参数，大约
  * 模型共包含 6 层transfomer，隐藏层包含 512 个单元。

## 有监督微调模型（Supervised finetuning）

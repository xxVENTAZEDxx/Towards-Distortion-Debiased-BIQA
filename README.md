# Towards Distortion-Debiased Blind Image Quality Assessment

[![ACM MM 2024](https://img.shields.io/badge/ACM%20MM-2024-blue)](https://doi.org/10.1145/3664647.3681704)
[![DOI](https://img.shields.io/badge/DOI-10.1145%2F3664647.3681704-green)](https://doi.org/10.1145/3664647.3681704)

**Publication**: ACM Multimedia 2024 

**Paper**: [DOI](https://doi.org/10.1145/3664647.3681704) 

**Authors**: Lize Zhou, Xiaoqi Wang, Jian Xiong, Xianzhong Long, Hao Gao 

---

## Project Overview | 项目简介

### English

This project is the official implementation of the paper **"Towards Distortion-Debiased Blind Image Quality Assessment"**. Existing Blind Image Quality Assessment (BIQA) models are susceptible to **Intensity Bias** and **Domain Bias**. Intensity bias refers to relatively accurate perception of severe distortions but larger estimation errors for mild distortions, while domain bias stems from discrepancies between synthetic and authentic distortion properties.

This work proposes a **unified debiasing learning framework** that integrates distortion perception and image restoration methods to mitigate intensity bias, and introduces a distortion domain recognition task to eliminate domain bias. The proposed method achieves state-of-the-art performance on multiple synthetic and authentic distortion datasets.

#### Key Features

- **Intensity Bias Mitigation**: Mildly distorted images are perceived through the restoration module generating reference images, while severely distorted images directly utilize the distortion perception branch
- **Domain Bias Elimination**: Introduces a domain recognition task based on intrinsic differences between distortion domains, using intra-domain similarity to weight quality scores
- **Intensity-Aware Cross-Attention**: Adaptively handles distortion biases of different intensities
- **Multi-Dataset Support**: Supports LIVE, CSIQ, TID2013, KADID-10K, LIVEC, KonIQ-10K, SPAQ, and other mainstream IQA datasets

### 中文

本项目是论文 **"Towards Distortion-Debiased Blind Image Quality Assessment"** 的官方代码实现。现有的盲图像质量评估（BIQA）模型容易受到**失真强度偏差**（Intensity Bias）和**失真域偏差**（Domain Bias）的影响。强度偏差表现为对严重失真的感知相对准确，但对轻微失真的估计误差较大；域偏差则源于合成失真与真实失真属性之间的差异。

本项目提出了一种**统一的去偏差学习框架**，通过整合失真感知与图像恢复方法来缓解强度偏差，并引入失真域识别任务来消除域偏差，在多个合成和真实失真数据集上达到了最先进的性能。

#### 核心特点

- **强度偏差缓解**：轻微失真图像通过恢复模块生成参考图像进行感知，严重失真图像直接利用失真感知分支
- **域偏差消除**：基于失真域固有差异引入域识别任务，利用域内相似度对质量分数进行加权
- **强度感知交叉注意力**：自适应处理不同强度的失真偏差
- **多数据集支持**：支持 LIVE、CSIQ、TID2013、KADID-10K、LIVEC、KonIQ-10K、SPAQ 等主流 IQA 数据集

## 📋 Table of Contents | 目录

- [Project Overview | 项目简介](#project-overview--项目简介)
- [Requirements | 环境要求](#requirements--环境要求)
- [Installation | 安装指南](#installation--安装指南)
- [Training | 训练流程](#training--训练流程)
- [Testing & Evaluation | 测试与评估](#testing--evaluation--测试与评估)
- [Configuration | 配置说明](#configuration--配置说明)
- [Project Structure | 项目结构](#project-structure--项目结构)
- [Pretrained Models | 预训练模型](#pretrained-models--预训练模型)
- [Citation | 引用](#citation--引用)



## Requirements | 环境要求

- **Python**: 3.8+
- **PyTorch**: 1.10+
- **CUDA**: 11.3+ (推荐使用 GPU 训练 | GPU recommended for training)
- **操作系统 | OS**: Windows / Linux

## Installation | 安装指南

### 1. Clone Repository | 克隆项目

```bash
git clone <your-repo-url>
cd iqa-project
```

### 2. Create Virtual Environment | 创建虚拟环境

```bash
conda create -n iqa python=3.9
conda activate iqa
```


## Training | 训练流程

This project adopts a **three-stage training strategy** | 本项目采用**三阶段训练策略**：

### Stage 1: Pretrain Classification Model | 阶段一：预训练分类模型

Pretrain the ViT classification branch on large-scale data to learn distortion feature representations. | 在大规模数据上预训练 ViT 分类分支，学习失真特征表示。

```bash
cd pretraining2
python train.py \
    --pretrain_path /path/to/pretraining/data \
    --label_path /path/to/labels \
    --batch_size 32 \
    --epochs 50 \
    --learning_rate 1e-4
```

**Output | 输出**：`class_model_best.pth` - Pretrained classification model weights | 预训练的分类模型权重

### Stage 2: Pretrain Image Restoration Model | 阶段二：预训练图像恢复模型

Train Swin Transformer UNet for image restoration. | 训练 Swin Transformer UNet 用于图像恢复。

```bash
cd pretraining1
python train.py \
    --pretrain_path /path/to/pretraining/data \
    --ref_path /path/to/reference/images \
    --label_path /path/to/labels \
    --batch_size 8 \
    --epochs 100 \
    --learning_rate 2e-5
```

**Output | 输出**：`checkpoint_model_best.pth` - Pretrained restoration model weights | 预训练的恢复模型权重

### Stage 3: Joint Training for Quality Assessment | 阶段三：联合训练质量评估模型

Load pretrained weights and perform end-to-end quality assessment training. | 加载预训练权重，进行端到端的质量评估训练。

```bash
# Move pretrained model weights to main directory | 将预训练模型权重移动到主目录
cp pretraining2/checkpoints/class_model_best.pth ./
cp pretraining1/checkpoints/checkpoint_model_best.pth ./

# Start training | 开始训练
python train.py \
    --dataset spaq \
    --path /path/to/spaq/dataset \
    --epochs 150 \
    --learning_rate 2e-5 \
    --train_bs 8 \
    --train_patch_num 1 \
    --test_patch_num 30
```

### Training Parameters | 训练参数说明

| Parameter | Default | Description |
| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--dataset` | `spaq` | Dataset name | 训练数据集名称 |
| `--path` | - | Dataset path | 数据集路径 |
| `--epochs` | 150 | Training epochs | 训练轮数 |
| `--learning_rate` | 2e-5 | Learning rate | 学习率 |
| `--weight_decay` | 1e-5 | Weight decay | 权重衰减 |
| `--train_bs` | 8 | Training batch size | 训练批次大小 |
| `--eval_bs` | 8 | Evaluation batch size | 评估批次大小 |
| `--train_patch_num` | 1 | Number of patches sampled during training | 训练时采样 patch 数量 |
| `--test_patch_num` | 30 | Number of patches sampled during testing | 测试时采样 patch 数量 |
| `--num_workers` | 4 | Number of data loading workers | 数据加载线程数 |
| `--seed` | 2035 | Random seed | 随机种子 |

## Testing & Evaluation | 测试与评估

### Single Dataset Testing | 单数据集测试

After training, the model will automatically evaluate on the test set and save results. Evaluation metrics include: | 训练完成后，模型会自动在测试集上评估并保存结果。评估指标包括：

- **SRCC** (Spearman Rank Order Correlation Coefficient)
- **PLCC** (Pearson Linear Correlation Coefficient)
- **RMSE** (Root Mean Square Error)

```bash
python train.py \
    --dataset live \
    --path /path/to/live/dataset \
    --checkpoints ./results/best_model.pth
```

### Cross-Dataset Evaluation | 跨数据集评估

```bash
python train_cross.py \
    --train_dataset live \
    --test_dataset1 csiq \
    --test_dataset2 tid2013 \
    --test_dataset3 kadid-10k \
    --test_dataset4 koniq-10k \
    --learning_rate 2e-5 \
    --epochs 70
```

### Cross-Dataset Configuration | 跨数据集配置

| Parameter | Default | Description |
| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--train_dataset` | `live` | Training dataset | 训练数据集 |
| `--test_dataset1` | `csiq` | Test dataset 1 | 测试数据集 1 |
| `--test_dataset2` | `tid2013` | Test dataset 2 | 测试数据集 2 |
| `--test_dataset3` | - | Test dataset 3 | 测试数据集 3 |
| `--test_dataset4` | - | Test dataset 4 | 测试数据集 4 |

## Configuration | 配置说明

All hyperparameters are managed through command-line arguments in `configs.py`: | 所有超参数通过 `configs.py` 中的命令行参数进行管理：

```python
# Optimizer Configuration | 优化器配置
--learning_rate 2e-5        # Learning rate | 学习率
--weight_decay 1e-5         # Weight decay | 权重衰减
--betas (0.9, 0.999)        # AdamW betas

# Data Loading Configuration | 数据加载配置
--train_bs 8                # Training batch size | 训练批次大小
--eval_bs 8                 # Evaluation batch size | 评估批次大小
--train_patch_num 1         # Training patch number | 训练 patch 数量
--test_patch_num 30         # Testing patch number | 测试 patch 数量
--num_workers 4             # Data loading workers | 数据加载线程数

# Training Configuration | 训练配置
--epochs 150                # Total training epochs | 总训练轮数
--start_epoch 1             # Starting epoch | 起始轮数
--seed 2035                 # Random seed | 随机种子
--train_test_round 2        # Train-test rounds | 训练-测试轮次
```

## Project Structure | 项目结构

```
iqaproject/
├── configs.py                              # Configuration file (hyperparameters) | 配置文件（超参数定义）
├── data_loader.py                          # Data loader (multi-dataset support) | 数据加载器（支持多数据集）
├── train.py                                # Main training script | 主训练脚本
├── train_cross.py                          # Cross-dataset training script | 跨数据集训练脚本
├── test.py                                 # Testing/utility script | 测试/工具脚本
├── swin_transformer_unet_skip_expand_decoder_sys.py  # Swin Transformer UNet
├── models/
│   └── networks.py                         # IQA model definition (core network) | IQA 模型定义（核心网络）
├── pretraining1/                           # Stage 1: Image restoration pretraining | 阶段一：图像恢复预训练
│   ├── data_loader.py
│   ├── metrics.py
│   ├── swin_transformer_unet_skip_expand_decoder_sys.py
│   └── train.py
├── pretraining2/                           # Stage 2: Classification pretraining | 阶段二：分类预训练
│   ├── classmodel.py
│   ├── data_loader.py
│   ├── metrics.py
│   └── train.py
├── results/                                # Training results output directory | 训练结果输出目录
└── checkpoints/                            # Model checkpoint directory | 模型检查点目录
```

## Pretrained Models | 预训练模型

The following model files will be generated after training: | 训练完成后会生成以下模型文件：

| File | Description | Location |
| 文件 | 说明 | 位置 |
|------|------|------|
| `class_model_best.pth` | Best classification pretrained model | 最佳分类预训练模型 | `pretraining2/checkpoints/` |
| `checkpoint_model_best.pth` | Best restoration pretrained model | 最佳恢复预训练模型 | `pretraining1/checkpoints/` |
| `best_model.pth` | Best quality assessment model | 最佳质量评估模型 | `results/` |

### Download Pretrained Models | 下载预训练模型

We provide pretrained model weights for direct use: | 我们提供预训练模型权重供直接使用：

- **Stage 1: Image Restoration Model** | 阶段一：图像恢复模型  
  [Download `checkpoint_model_best.pth`](https://drive.google.com/file/d/1xaNYR-nLP0Vx8OBz6h_VPC41RQSUJVy3/view?usp=drive_link)

- **Stage 2: Classification Model** | 阶段二：分类模型  
  [Download `class_model_best.pth`](https://drive.google.com/file/d/1SpLkj6nheT2Ya8kClzkCPdkzBVRW7oe_/view?usp=drive_link)

## Citation | 引用

If you use this code in your research, please cite: | 如果你在项目研究中使用了本代码，请引用：

```bibtex
@inproceedings{Zhou2024TowardsDB,
  title={Towards Distortion-Debiased Blind Image Quality Assessment},
  author={Lize Zhou and Xiaoqi Wang and Jian Xiong and Xianzhong Long and Hao Gao},
  booktitle={Proceedings of the 32nd ACM International Conference on Multimedia},
  year={2024},
  url={https://doi.org/10.1145/3664647.3681704}
}
```
```

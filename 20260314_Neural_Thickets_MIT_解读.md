# 论文解读：Neural Thickets: Diverse Task Experts Are Dense Around Pretrained Weights
> MIT CSAIL | arXiv:2603.12228 | 2026年3月12日发布 | 开源地址：https://github.com/sunrainyg/RandOpt

## 核心突破
首次发现大模型存在**"神经丛林"（Neural Thickets）现象**：
- 小模型：权重周围是"大海捞针"状态，找到优秀解需要梯度下降等结构化优化
- 7B+大模型：预训练权重周围密集分布着大量任务专属专家，随机采样就能得到性能更优的解，不需要迭代优化

## 极简算法 RandOpt
仅需两步就能实现和PPO、GRPO等主流后训练方法相当的效果：
1. **随机采样：** 对预训练权重加N个高斯扰动
2. **Top K集成：** 选效果最好的K个模型结果投票输出
✅ 完全并行，200 GH200集群上7B模型训练仅需3.2分钟

## 实测效果
- **LLM：** Olmo-3-7B在Countdown任务上准确率达到70%
- **VLM：** Qwen2.5-VL-3B在GQA视觉推理数据集上准确率提升12.4%（56.6% → 69.0%）
- **通用性：** 在数学推理、代码生成、创意写作、化学合成等7类任务上均超过基线

## 核心配图
### 图1：大小模型权重空间景观对比图
![图1](https://arxiv.org/html/2603.12228/x1.png)
### 图2：0.5B-32B模型权重空间精度热力图
![图2](https://arxiv.org/html/2603.12228/x2.png)
### 图3：解密度/多样性随模型尺寸缩放定律
![图3](https://arxiv.org/html/2603.12228/x3.png)
### 图4：随机采样专家的任务专业化特性
![图4](https://arxiv.org/html/2603.12228/x4.png)
### 图6：RandOpt vs PPO/GRPO/ES性能对比
![图6](https://arxiv.org/html/2603.12228/x6.png)

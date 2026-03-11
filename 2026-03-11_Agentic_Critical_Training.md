# 前沿论文解读：Agentic Critical Training（ACT）——让AI Agent真正学会批判性思考

## 1. 论文基本信息
- **标题**：Agentic Critical Training
- **发布时间**：2026年3月10日
- **作者团队**：马里兰大学帕克分校 Weize Liu、Minghui Liu 等
- **领域**：AI Agent、强化学习、大语言模型
- **论文链接**：https://arxiv.org/abs/2603.08706
- **代码仓库**：https://github.com/ajayn1997/Self-Critical-Sequential-training-with-RL-for-chatbots

## 2. 研究背景与问题提出
当前LLM Agent训练主要依赖模仿学习（Imitation Learning，IL），通过在专家演示数据上微调让模型复制成功行为，但这种方法存在本质缺陷：
1. 只教Agent“做什么”，但不教“为什么这么做是对的”、“什么是错误的行为”
2. 遇到训练集中没有的次优状态或失败场景时，Agent无法自适应调整
3. 现有自反思方法只是让模型模仿预先生成的反思文本，没有真正形成自主的批判性思考能力

针对上述问题，论文提出了**Agentic Critical Training（ACT）** 强化学习范式，训练Agent自主区分优质和劣质动作，形成真正的批判性推理能力。

## 3. 核心技术方案
ACT是一个两阶段的强化学习训练 pipeline，针对部分可观测马尔可夫决策过程（POMDP）场景下的LLM Agent设计：

![ACT两阶段训练架构图（ArXiv官方原始图）](images/2026-03-11/figure1_arch.svg)

### 3.1 数据构造阶段
对于训练集中的每个专家状态-动作对 $(s_i, a_i)$，构造对比样本：
- 基于初始策略生成多个替代动作作为负样本
- 将专家动作与每个替代动作配对，形成对比训练样本
- 核心假设：初始策略生成的动作通常劣于专家动作，可以作为有意义的负例用于判别学习

### 3.2 两阶段训练流程
**阶段1：Agent批判性训练**
- 模型学习在专家动作和替代动作候选中识别更优的动作
- 仅提供二进制反馈（是否正确识别了专家动作），不提供显式的推理文本监督
- 强制模型自主发展思维链推理能力来最大化奖励

**阶段2：RL动作训练**
- 在完成批判性训练后，在专家轨迹上进行标准强化学习，直接生成动作
- 利用阶段1形成的批判性推理能力指导动作生成

### 3.3 奖励函数设计
采用分组相对策略优化（GRPO），复合奖励函数：
$$R(s, y) = R_{acc} + R_{adm} + R_{fmt}$$
- $R_{acc}$：完全匹配专家动作时获得全额奖励
- $R_{adm}$：对于可接受但次优的动作给予部分奖励
- $R_{fmt}$：惩罚不正确的输出格式

## 4. 实验结果与关键发现
研究在三个不同的Agent基准测试集上验证了ACT的效果：ALFWorld（具身任务）、WebShop（网页导航）、ScienceWorld（科学实验），使用Qwen3-8B和Qwen3-4B模型。

![各基准测试集性能对比结果（ArXiv官方原始图）](images/2026-03-11/figure2_results.svg)

### 4.1 性能提升显著
- 作为预训练阶段使用时，ACT比单纯模仿学习平均性能提升5.07个百分点，比单纯强化学习平均提升4.62个百分点
- 优于现有基于反思的方法“Early Experience”平均2.42个百分点
- 「RL + ACT」组合取得了整体最优性能

### 4.2 泛化能力增强
- 在分布外（OOD）任务上提升更明显：ALFWorld的OOD分割集上，ACT给RL带来3.73个百分点的提升，而分布内任务提升为2.15个百分点
- 说明批判性推理能力能有效迁移到新场景

### 4.3 故障恢复能力
定性分析显示：
- ACT训练的模型遇到动作失败时，会进行自我诊断、识别根本原因并执行纠正动作
- 传统IL训练的模型会反复尝试相同的失败动作，缺乏纠错能力

### 4.4 通用推理能力迁移
令人意外的发现：在Agent任务上的ACT训练，无需任何推理专用训练数据，就能提升通用推理基准测试的性能：
- 传统IL训练会导致“推理坍塌”，在GPQA-Diamond基准上性能下降6.91个百分点
- ACT训练相比基线prompt方法性能提升1.85个百分点
- 分析显示ACT模型自发发展出了自我验证行为，会系统地将答案代回原方程检查，消除不一致的选项

## 5. 工程启示与落地价值
1. **训练范式革新**：ACT打破了模仿学习的局限性，证明批判性推理是可训练的能力，而非仅靠prompt引导
2. **部署可靠性提升**：故障恢复能力和分布外性能提升，让ACT训练的Agent更适合真实世界部署（会频繁遇到意外情况）
3. **多任务收益**：一次ACT训练能同时提升Agent任务性能和通用推理能力，训练性价比更高
4. **现有系统适配**：ACT作为预训练阶段可以无缝集成到现有的Agent训练流程中，无需大幅改造架构

## 6. 研究局限性与未来方向
- 目前仅在8B/4B规模模型上验证了效果，更大规模模型上的表现有待验证
- 负样本构造依赖初始策略的质量，未来可探索更优的负例生成方法
- 可研究将ACT与其他Agent技术（如工具使用、记忆模块）结合的效果
- 可探索ACT在多Agent协作场景下的应用

## 7. 相关参考资料
- [Agent learning via early experience](https://arxiv.org/abs/2510.08558)：ACT对比的基线方法
- [Deepseekmath: Pushing the limits of mathematical reasoning in open language models](https://arxiv.org/abs/2402.03300)：GRPO强化学习算法的来源
- [Reflexion: Language agents with verbal reinforcement learning](https://arxiv.org/abs/2303.11366)：相关的自反思Agent研究
- [React: Synergizing reasoning and acting in language models](https://arxiv.org/abs/2210.03629)：LLM Agent推理-行动范式的基础工作

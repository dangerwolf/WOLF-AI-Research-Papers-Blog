+++
date = '2026-07-17T14:30:20+08:00'
draft = 'false'
title = 'A Benchmark and Framework for Evaluating Next Action Predictions in Spreadsheets'
tags=['SpreadSheet', 'Benchmark', 'Prediction']
categories = ['SpreadSheet']

+++





[TOC]





[总体解构](#overview) / [实验细节解构](#exp-details) / [AI中文讲解](#ai-zh) / [AI英文讲解](#ai-en)

| ITEM    | METADATA                                                     |
| ------- | ------------------------------------------------------------ |
| title   | A Benchmark and Framework for Evaluating Next Action Predictions in  Spreadsheets |
| date    | 2026-07-17                                                   |
| venue   | ICML 2026                                                    |
| authors | ["Tejas Agrawal", "Vu Le",  "Sumit Gulwani", "Gust Verbruggen"] |
| year    | 2026                                                         |
| pdf     | https://arxiv.org/abs/2606.13802                             |
| code    | www.github.com/Tej-55/NAPE                                   |
| tags    | []                                                           |

针对电子表格缺乏自动补全功能的痛点，本文提出了首个电子表格动作预测基准及交互式评估框架，通过构建数据集和多种模型对比，探讨了实现智能化电子表格交互的可行性与影响因素。





# 总体解构 {#overview}



---

## 1. 【核心痛点】

* **一句话定义**：这篇论文试图解决**如何在没有真实历史修改记录、且操作空间极其复杂的多维电子表格中，评测和构建“无模态”（Modeless）的下一步操作主动预测系统**。


* **前人困境**：
* **数据缺失**：公开的电子表格数据集（如 Enron）只有静态的最终版，完全没有记录用户一步步搭建表格的“动态轨迹（Edit Histories）”。


* **评测失真（思路错误）**：传统评测多采用**教师逼近式离线评估（Teacher-forced Offline Evaluation）**，将每一步隔离开进行静态匹配。这种做法无法模拟真实人机交互中“接受预测会导致后续操作发生改变”、“错误预测需要用户撤销/修正从而产生额外开销”的动态累积效应。





---

## 2. 【解题机制】

* **核心直觉**：**“他把‘评测代码自动补全’看作了‘动态强化学习的 On-policy Rollout 模拟游戏’”**。系统不只看预测得对不对，而是直接扮演用户，动态演进未来 ground truth 的轨迹。


* **关键步骤（神来之笔）**：
* **动态轨迹重塑（Dynamic Ground-truth Adaptation）**：当模拟用户接受了一个包含部分错误（False Positive）的预测时，评估框架不会宕机，而是**动态修改剩余的 ground-truth 动作序列（Future）**——它会自动在未来的头部插入“撤销操作（Inverse Operations）”来抹平模型的错误，并计算模型真正帮用户省去了多少次手动点击（User Actions Saved, UAS）。


* **三阶段轨迹逆向工程**：通过“VLM语义标注（Region Map） $\rightarrow$ 启发式冷启动生成（多参数融合排序） $\rightarrow$ LLM 裁判-编辑器双重纠偏 $\rightarrow$ 人工最终校对”的管线，从静态表格中反向逼真重建了 52 个任务、共 12,000 步的高质量人类编辑轨迹。





---

## 3. 【创新增量】

* **对比**：
* 相比于 **SOTA/主流的表格 Assistant**（如 SheetCopilot）：从依赖用户显式输入自然语言指令（NL-to-action）的被动式对话，转向了**静默观察用户历史并主动推荐（Action-to-action）的无模态自动补全**。


* 相比于 **静态 Offline 评测**：引入了 **Online 交互评测指标**（如结合了惩罚机制的净节省步数比率 UAS），能真正测出哪些预测是在“帮倒忙”。




* **本质**：为学术界和工业界贡献了**首个电子表格下一步操作预测基准数据集（NAPE）** 与一套**能够自我纠偏、闭环运行的交互式模拟评测协议**。



---

## 4. 【批判性边界】

* **隐形假设**：
* **完美理性的用户接受模型**：实验中定义的用户接受启发式（如 GREEDY、P90 等）假设人类在面对错杂的推荐时，能瞬间且无认知过载地判断出该推荐的正确率并做出 Accept/Reject 决定。实际上，人类阅读并审查 15 个连续填充格子的视觉成本可能高于直接手动输入。


* **原子操作的可逆性**：假设所有 false positive 操作都可以通过一个简单的逆操作（如 `clear border`）无缝回滚且不干扰其他未污染区域。




* **未解之谜**：
* **主动弃权（Abstention）机制的缺失**：在没有人类 heuristic 过滤时（如 ALWAYS 模式），主流 LLM 依然倾向于“过度预测”和“不知道什么时候该闭嘴”，导致净节省步数变为灾难性的 $-19.2\%$。模型自身如何感知不确定性并主动拒绝预测，依然是未解之谜。


* **冷启动困境**：在表格构建的前 $10\%$ 进度中，由于缺乏上下文规律，所有模型的预测接受率都极低。





---

## 5. 【一言以蔽之】

### napkin 示意图

```text
           [ 当前状态 S_t ]  -----( 输入历史 H_t )-----> [ 预测器 Solver ]
                  |                                           |
             (执行用户动作)                               (生成预测 P_t)
                  |                                           |
                  v                                           v
           [ 用户实际动作 ] <---( 动态注入反向擦除/补齐 )--- [ 模拟用户决策 Accept? ]
                  |                                           |
                  +===================[ 动态更新剩余 Future 序列 ]

```

### 核心公式

衡量该任务成败的终极指标是 **用户动作节省率（UAS）**：

$$UAS = \frac{L_{\text{initial}} - L_{\text{final}}}{L_{\text{initial}}}$$

其中 $L_{\text{initial}}$ 为无辅助下完成表格所需的纯手动步骤总数，$L_{\text{final}}$ 为在预测器辅助下（包含模型犯错导致用户被迫执行额外撤销、修正操作后）实际发生的人类动作总数。

---





# 实验细节解构 {#exp-details}





## 1. 数据集详情 (Dataset)

-   **数据集名称**：**NAPE (Next Action Prediction in Spreadsheets)**。
-   **出自哪里**：由微软 PROSE 研究团队（作者包括知名学者 Sumit Gulwani 等）人工策划并反向构建。原始表格的逻辑和样式种子来源于已有的**公开电子表格语料库**（如著名的 Enron 数据集等）。由于公开语料只存有最终静态表格而无用户修改历史，作者团队使用“启发式冷启动生成 + LLM 裁判-编辑器纠偏 + 人工校验”的管线，重构出了用户一步步搭建表格的动态轨迹。
-   **是否开源**：**是**。
-   **是否可以下载**：**是**。
-   **下载地址**：
    -   GitHub 仓库：https://github.com/Tej-55/NAPE。
-   **数据集体量**：包含 **52 个高质量的电子表格创建轨迹（Trajectories）**，涵盖数据输入、公式计算、边框、颜色和合并单元格等多维操作，总共包含 **11,907 个原子操作步骤**（均通过人工最终校对验证）。

## 2. 对比方法与目标 (Baselines & Targets)

本研究对比了多类不同量级和技术路线的预测器（Solvers）：

| **预测方法分类**                       | **具体模型/目标 (Baselines)**                                | **出自哪里 / 是否开源 / 下载地址**                    | **选择这些方法/目标的原因**                                  |
| -------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| **闭源闭端大模型 (Reasoning LLMs)**    | **GPT-5** 等前沿推理模型                                     | 微软/OpenAI 闭源，通过 API 商业化付费使用。           | 作为当前大模型能力的上界（Upper Bound），测试大模型在**长上下文和复杂空间逻辑**推理下的代码补全极限。 |
| **轻量级闭源模型 (Mini LLMs)**         | **GPT-5 mini** 等轻量级大模型                                | 微软/OpenAI 闭源，通过 API 商业化付费使用。           | 探究在低延迟、高性价比场景下，轻量大模型的常态推理表现。     |
| **自监督微调小模型 (Finetuned SLMs)**  | 经过自监督微调的 **360M 参数级** 小语言模型（Small Language Model） | **开源**，微调代码及方法托管于上述 NAPE 官方 GitHub。 | 验证在垂直领域（电子表格操作）通过微调，小模型（SLM）是否能在特定任务上**媲美/超越大模型（以小博大）**。 |
| **经典基准方法 (Classical Baselines)** | **启发式/统计规则方法**（如特定模式的常态复制、自动填充填充法） | **开源**，在 NAPE 仓库中提供内置基准代码。            | 提供非黑盒的、可解释的经典基准线，用以证明引入 LLM 的必要性。 |

## 3. 实验评估指标 (Evaluation Metrics)

### 3.1 实验的归属类别

本实验属于“交互式序列生成与预测（Next Action Prediction with Interactive Rollout / Simulator）”。它不仅是一个静态的“输入 X 预测 Y”的单步预测，而是一个引入了“模拟用户（Simulated User）”反馈的、**动态在线模拟交互评估（On-policy Evaluation）**。

### 3.2 行业通用评估指标与局限

-   **行业传统评估指标**：**Top-k 准确率（Accuracy）、精确率（Precision）、召回率（Recall）**。
-   **传统指标的缺陷**：传统的“教师逼近离线评估（Teacher-forced Offline Evaluation）”会强行假设用户不接受任何带错误的预测。然而在现实中，模型生成了 10 个动作，用户可能接受了其中 8 个，而手动撤销了 2 个。这种“动态纠偏”造成的连锁反应和额外撤销开销，传统静态指标完全无法衡量。

### 3.3 论文提出的新型评估指标（及作者的选择理由）

为了模拟真实的物理人机交互，作者设计了以下指标：

-   **用户动作节省率 (User Actions Saved, UAS)** （核心指标）：

    $$UAS = \frac{L_{\text{initial}} - L_{\text{final}}}{L_{\text{initial}}}$$

    -   $L_{\text{initial}}$：人类纯手动构建表格的总步数。
    -   $L_{\text{final}}$：有 AI 辅助下（包括用户执行 AI 的推荐、以及**撤销 AI 推荐中错误步骤所花费的惩罚步数**）人类实际执行的总动作数。
    -   **选择理由**：它真实反映了 AI 能否切实减少人类的点击次数，避免了“AI 瞎预测导致人类频繁撤销，反而增加工作量”的虚假繁荣。

-   **接受率 (Acceptance Rate, AR)**：模拟用户接受预测的比例。

-   **预测精度 (Average Precision, Prec)**：宏观精确度，评估推荐列表中正确操作的密度。

-   **可预测性覆盖率 (Predictability Coverage, pCov)**：评估系统捕获到了多少理论上可以通过上下文推断出来的操作（剔除了由于新冷启动导致的不可预测步骤，使指标对模型能力的衡量更公平）。

## 4. 评估实验的 RQ 逻辑链条与批判

论文围绕以下几个研究问题（Research Questions, RQs）展开，其逻辑链条与我的审稿人视角的推演如下：

Plaintext

```
               ┌─────────────────────────────────────────────────────────┐
               │  [RQ1] 复杂模型是否优于简单模型？ ──────> 是 (GPT-5 胜出)     │
               └────────────────────┬────────────────────────────────────┘
                                    │ (但推理成本极高)
                                    v
               ┌─────────────────────────────────────────────────────────┐
               │  [RQ2] 小模型微调能否逼近大模型？ ───> 是 (360M 媲美 GPT-5)  │
               └────────────────────┬────────────────────────────────────┘
                                    │ (说明任务可学，但模型总是胡乱预测)
                                    v
               ┌─────────────────────────────────────────────────────────┐
               │  [RQ3] 拒识(Abstention)与触发机制 ─> 极其关键 (无拒识会导致 UAS 变负)│
               │        对最终体验的影响？                                 │
               └─────────────────────────────────────────────────────────┘
```

### 4.1 思维逻辑链分析

-   **逻辑链一：能力上界确立 (RQ1: Model Scaling)**：首先测试最强的 LLM（GPT-5）与轻量 LLM 在此任务上的 UAS 表现。证明了在无模态自动补全中，大模型的推理和空间规划能力能带来直接的 UAS 提升。
-   **逻辑链二：实用性落地 (RQ2: SLM Finetuning)**：大模型虽然强，但在 Excel 这种需要毫秒级低延迟的场景中不可行。因此探讨微调 360M 小模型，发现微调后其表现（UAS ~27%）能够直接追平 GPT-5。
-   **逻辑链三：边界与约束 (RQ3: Acceptance & Triggers)**：当模型不断预测时，人会烦。实验测试了“在不同触发频率（如每 1 步预测一次 vs. 每 4 步预测一次）”以及“无条件接受 vs. 带不确定性拒识（Abstention）”下的表现。结果证明：**如果模型不具备“在不确定时闭嘴（Abstention）”的能力，UAS 会直接暴跌至 $-19.2\%$（反而给用户添堵）**。

### 4.2 合规、合理、合辑性评估

-   **优点（极为合理）**：整个逻辑链闭环且极度务实。它不仅追求学术上的“准确率提高”，还深入到了端侧部署、用户体验（Abstention 机制）和延迟（小模型微调）的工业落地痛点，是非常扎实的系统级工作。
-   **如果是你（我），是否会进行调整？**
    -   *调整建议*：我会增加 **“多模态/视觉干扰（Visual Noise）对用户接受度的影响（RQ4）”**。在实际 Excel 操作中，用户接受/拒绝推荐，不仅取决于推荐的逻辑对不对，还取决于**视觉 UI 呈现的方式**。如果模型一次性推荐了 15 个格子的填充，人类的视觉扫描和审查成本（Cognitive Load）其实极高。论文的模拟用户机制（Heuristics）过于理论化，没有量化“人类审查 AI 预测所需的脑力开销”。

## 5. 实验细节与注意事项 (Experimental Details)

在复现和运行 NAPE 框架时，必须注意以下系统设计细节（避坑指南）：

1.  **动态路径修剪（Ground-truth Adaption）**：

    当模拟用户接受了一个包含 False Positive（FP，即部分错误）的预测后，评估框架会自动计算“逆向操作（Inverse Operations）”，并将这些逆向操作动态插入到接下来的 Ground-truth 队列头部。在编写自定义 Solver 时，**必须确保你的模型能正确识别并处理由框架动态补入的、与原编辑历史不同的状态**。

2.  **原子操作可逆性假定**：

    框架默认大部分基础操作（如 `set_value`）的逆操作（如 `clear_value`）是直接等价的。在引入复杂自定义操作（如复杂的宏合并、跨表联动）时，需要额外定义可逆映射函数，否则会导致状态死锁。

3.  **大模型调用的状态窗口**：

    由于是 Online Rollout 模式，单次表格构建可能需要调用 LLM 数百次（每一步都要预测）。复现时需注意 API 速率限制（Rate Limit）与高昂的 API 账单成本，建议在本地部署微调后的 SLM 进行调试。

## 6. 论文贡献 (Contributions)

论文的核心学术和技术贡献可总结为以下三点：

1.  **首个开源动态轨迹数据集 (NAPE Dataset)**：手动策划并校验了 52 个高质量、全生命周期的电子表格构建动作轨迹（共计 1.2 万步），填补了表格领域“只有最终静态状态，没有动态编辑历史”的空白。
2.  **创新的在线模拟评测框架 (Online Simulator)**：摒弃了传统的静态离线测试，推出了首个在评测中能够**根据模型预测动态改写未来 Ground-truth 轨迹、允许撤销和纠偏**的闭环评估框架，并定义了极具现实指导意义的 **UAS 指标**。
3.  **设计方法学启示 (Design Insights)**：通过大量 Baseline 实验，给工业界研发 Spreadsheet Copilot 提供了极为重要的设计启示：**模型的“拒识能力（Abstention）”和“轻量化部署与精准触发（Cheap Triggers）”，比一味盲目追求扩大模型参数量要重要得多**。



---

# AI中文讲解 {#ai-zh}

## 微软这回整了个大活：他们给AI装了个“记账本”，专治表格界的多动症

如果你日常工作需要和 Excel 打交道，你一定经历过这种被支配的恐惧： 本来只想拉个简单的数据表，结果一会儿要调字体颜色，一会儿要加个边框，一会儿又要写个复杂的 `VLOOKUP` 公式。手鼠标和键盘之间疯狂切换，感觉自己像个在厨房里同时炒八个菜的厨师，手忙脚乱。

这时候你可能会想：“现在大模型这么厉害，能不能像写代码的自动补全一样，我刚在 Excel 里填了两个格子，AI 就能猜出我下一步想干嘛，直接帮我把剩下的活儿干了？”

想法很美好，但直到最近，整个科技界其实都拿这件事情没办法。**因为在电子表格这个领域，AI 一直处于一种“半盲”的状态。**



为了解决这个老大难问题，计算机软件领域的顶级殿堂——微软 PROSE 研究团队（由大名鼎鼎的学者 Sumit Gulwani 领衔），直接在 **ICML 2026** 国际机器学习大会上甩出了一项颠覆性的研究成果。他们不仅开发了一套全新的测试框架 **NAPE**，还揭示了如何让 AI 真正变成一个贴心的“表格副驾驶”。

今天，我们就来拆解一下，科学家们是如何给 AI 设下“陷阱”，逼它学会看懂人类在表格里的每一步骚操作的。

## 致命的痛点：AI 根本不知道你曾经“爱过”

要让 AI 预测你的下一步，它首先得知道你前几步做了什么。这听起来是常识，但在电子表格的世界里，这简直是奢望。

平时我们看到的公开表格数据集（比如著名的 Enron 数据集），全都是“最终完成版”。这就像你手里拿着一本已经印好的小说，但你根本不知道作者在写作过程中，删掉过哪段话、改过哪个错别字、中间停顿了多久。AI 拿到的只有静态的结果，完全没有“历史修改轨迹”。

更糟糕的是，以前科学家们测试 AI 的方法非常“填鸭式”（学术界叫教师逼近式离线评估）。 他们把表格切成片段喂给 AI：“看，这步我填了A，你猜下一步是什么？”AI 猜完，不管对错，科学家立刻把正确答案塞给它，继续问下一步。

但这根本不符合我们人类的操作逻辑！在真实世界里，如果 AI 给你预测了一个错误的操作，你把它顺手接受了，那表格的数据就已经被污染了。接下来你必须先执行“撤销”，或者换个方式去修补它。传统测试方法完全忽略了这种“AI 犯错、人类擦屁股”的连锁动态效应。

## 破案思路：把“表格补全”变成一场“动态通关游戏”

微软的研究团队一拍大腿：既然没有历史记录，那我们就自己**逆向工程**，把一张死表格“复活”成一段动态的搭建过程！

他们动用了多模态大模型和各种精密的算法，把 52 个高质量表格的诞生过程，像放电影一样倒带，还原出了 **11,907 个原子操作步骤**。这里面包含了人类填数据、写公式、画边框、改颜色的每一步细节。这个被复活的宝贵数据集，就是 **NAPE**。

接着，他们设计了一个极其聪明的“交互式模拟器”。这个模拟器不再是死板的填鸭式教学，而是变成了一场游戏：

-   模拟器扮演“人类用户”，大模型扮演“AI 助手”。
-   AI 给出下一步的预测推荐。
-   “人类用户”根据一套聪明的规则来决定是接受还是拒绝。
-   **最绝的地方来了**：如果 AI 推荐的步骤里包含了一点小错误，而“人类用户”不小心接受了，模拟器不会卡死，它会**立刻启动“动态纠偏”机制**——在接下来的任务清单里，动态插入几步“擦除/撤销”操作，先把 AI 捅的娄子补上，然后继续往下演进！

通过这种方式，科学家们终于能算出一笔铁账：**AI 到底是帮了忙，还是在帮倒忙？**



他们提出了一个终极核心指标：**用户动作节省率（User Actions Saved, UAS）**。 简单来说，如果平时做这个表需要点 100 下鼠标，现在有了 AI，你只需要点 70 下（哪怕其中有 5 下是用来撤销 AI 错误预测的），那这个 AI 的 UAS 就是 $30\%$。如果 UAS 变成负数，对不起，这个 AI 就是个纯粹的“猪队友”。

## Aha! 时刻：不会闭嘴的 AI，就是灾难

科学家们把当时最顶尖的闭源推理大模型（如 GPT-5）、轻量级小模型，以及通过自监督微调的 360M 参数专用小模型（SLM）全部拉进这个模拟器里进行了一场大比武。

实验结果揭示了两个让人直呼“哇塞”的重大发现：

### 1. 以小博大：360M 小模型追平万亿大模型

在以前的认知里，大模型参数量越大越聪明。但微软发现，在电子表格预测这个特定垂直赛道里，经过自监督微调的 **360M 参数小模型**，其节省步数的表现（UAS 约为 $27\%$）居然能够直接**追平甚至超越那些体量庞大、价格昂贵的顶级大模型**！这意味着，未来我们完全可以在本地电脑、甚至手机端离线运行一个极其敏捷的表格助手，不需要昂贵且高延迟的网络 API。

### 2. 反直觉真相：不知道什么时候该“闭嘴”的 AI，杀伤力极大

这是全篇论文最精彩的发现。很多人觉得，AI 嘛，预测得越多越好，反正错了我拒绝就行了。 但实验证明，如果让 AI 开启“无条件疯狂预测”模式，它的 UAS 指标会直接暴跌到 **$-19.2\%$**！ 因为大模型都有“过度自信”的毛病，它不知道自己不知道。当它疯狂弹出错误的预测时，模拟用户哪怕只是频繁地去点击“拒绝”或者“撤销”，累积下来的工作量就已经远远超过了自己从头手打表格的开销！

>   **结论很明确**：一个优秀的表格 AI，最核心的超能力不是“有多聪明”，而是**“知之为知之，不知为不知”的拒识能力（Abstention）**。在没把握的时候，保持静默，才是对人类最大的温柔。

## 所以呢？它将如何改变我们的未来？

微软 PROSE 团队的这项研究，相当于为全球的办公软件开发者提供了一把高度精准的尺子。

在过去，各家厂商在做“智能表格”时，大多停留在“被动响应”阶段——你输入一句“帮我把这一列求和”，它动一下。这种无模态的“主动预测”因为缺乏评测标准，大家都在黑灯瞎火里摸索。

而现在，**NAPE 框架彻底开源了**。它告诉所有人：

1.  不要盲目卷大模型参数，在端侧把小模型调教好，反应又快又省电。
2.  必须重点研发“不确定性触发机制”，让 AI 学会评估自己的信心，只在有八九成把握的时候才跳出来惊艳用户，其余时间乖乖当个隐形人。

也许在不久的将来，当你再次打开 Excel 准备大干一场时，你会发现那个静默在侧的 AI 变得无比默契。你刚打出两个部门的名称，它就已经默默为你铺好了整张报表的骨架，而你只需要轻松地敲一下 `Tab` 键表示赞同。

这才是科技该有的样子：润物细无声，把人类从繁琐的“多动症”操作中，彻底解放出来。

---

# AI英文讲解 {#ai-en}



## Microsoft has pulled off a big feat this time: they’ve equipped AI with a "notebook" to cure the hyperactivity of spreadsheets.



If your daily work involves dealing with Excel, you must have experienced that overwhelming fear: you originally just wanted to create a simple data table, but soon you're adjusting font colors, adding borders, and writing complex `VLOOKUP` formulas. You find yourself frantically switching between mouse and keyboard, feeling like a chef trying to cook eight dishes at once in the kitchen—completely flustered.



At this point, you might think: “Now that large models are so powerful, can’t they guess what I want to do next after I fill in two cells in Excel? Can’t AI just help me finish the rest of the tasks directly?”



It’s a nice thought, but until recently, the entire tech industry had no solution for this issue. **Because in the realm of spreadsheets, AI has always been in a state of 'semi-blindness.'**



To tackle this longstanding problem, Microsoft's PROSE research team—a top-tier institution led by renowned scholar Sumit Gulwani—unveiled a groundbreaking research achievement at the **ICML 2026** International Conference on Machine Learning. They not only developed an entirely new testing framework called **NAPE**, but also revealed how to make AI truly become an attentive "co-pilot" for spreadsheets.



Today we will break down how scientists set up “traps” for AI to force it to learn how to understand every step humans take within spreadsheets.



## The Critical Pain Point: AI Has No Idea What You’ve Previously "Loved"



For AI to predict your next move, it first needs to know what actions you've taken previously. This sounds like common sense; however, it's almost wishful thinking in the world of spreadsheets.



The publicly available spreadsheet datasets we usually see (like the famous Enron dataset) are all “final versions.” It’s akin to holding a printed novel without knowing which passages were deleted during writing or which typos were corrected along the way—the static results provided leave out any “historical modification traces.”



Even worse is that previous methods used by scientists for testing AI were very much “cramming-style” (known as teacher-approximated offline evaluation in academia). They would slice up tables into segments and feed them into AI: “Look here; I filled A here; now guess what comes next?” After making its guess—right or wrong—the scientist would immediately provide it with the correct answer and continue asking about subsequent steps.



But this completely disregards our human operational logic! In reality if an AI predicts an incorrect action and you accept it casually then data within that spreadsheet becomes corrupted. Next you'll need either undoing or some other method of fixing it. Traditional testing methods completely overlook this chain dynamic effect where "AI makes mistakes while humans clean up.



## Case Solving Approach: Turning "Table Completion" into a "Dynamic Clearance Game"



Microsoft's research team had an epiphany: since there are no historical records, let's **reverse engineer** and bring a static table back to life as a dynamic construction process!



They employed multimodal large models and various sophisticated algorithms to rewind the birth process of 52 high-quality tables like playing a movie, restoring **11,907 atomic operation steps**. This included every detail of human data entry, formula writing, border drawing, and color changing. This revived valuable dataset is called **NAPE**.



Next, they designed an extremely clever “interactive simulator.” This simulator transformed from rigid rote teaching into a game:



*   The simulator acts as the “human user,” while the large model plays the role of the “AI assistant.”

*   The AI provides predictions for the next step.

*   The “human user” decides whether to accept or reject based on a set of smart rules.

*   **The most remarkable part:** If there’s even a small error in one of AI’s recommended steps that the “human user” accidentally accepts, instead of freezing up, the simulator will **immediately activate its "dynamic correction" mechanism**—dynamically inserting several "erase/undo" operations in subsequent task lists to fix what went wrong before continuing!



Through this method, scientists could finally calculate an ironclad account: **Is AI truly helpful or actually counterproductive?**



They proposed an ultimate core metric: **User Actions Saved (UAS)**. Simply put, if normally it takes 100 mouse clicks to complete this table but with AI you only need 70 clicks (even if 5 are used for undoing AI's erroneous predictions), then this AI has a UAS of $30\%$. If UAS becomes negative—sorry! That AI is just a pure "bad teammate."



## Aha! Moment: An Unquiet AI Is Disastrous



Scientists brought together top-tier closed-source reasoning large models (like GPT-5), lightweight small models, and self-supervised fine-tuned specialized small models with 360M parameters (SLM) for an extensive competition within this simulator.



The experimental results revealed two astonishing discoveries:



### 1. Small vs Big: The 360M Small Model Matches Trillion Parameter Large Models



In previous understanding, larger parameter counts meant smarter models. However, Microsoft found that in the specific vertical domain of spreadsheet prediction after self-supervised fine-tuning; their **360M parameter small model**, which saved steps with performance (UAS around $27\%), was able to directly match or even surpass those massive and expensive top-tier large models! This means we can potentially run an extremely agile spreadsheet assistant locally on our computers or even offline on mobile devices without needing costly high-latency network APIs.



### 2. Counterintuitive Truth: An Unaware-to-Be-Silent AI Can Be Highly Destructive



This was the most exciting finding in the entire paper. Many people think that more predictions from AI are better; after all I can just reject any errors it makes. But experiments showed that when allowed to operate under “unconditional crazy prediction” mode, its UAS would plummet directly down to **$-19.2\%$**! Because large models tend toward being overly confident—they don’t know what they don’t know. When they wildly generate incorrect predictions repeatedly popping up during simulation interactions—even frequent clicking on "reject" or "undo," accumulates work far exceeding simply entering data manually from scratch!



>   **The conclusion is clear:** A great spreadsheet AI's core superpower isn’t how smart it is but rather its ability for refusal recognition (**Abstention**)—knowing when not to speak up when uncertain keeps silence at times being humanity’s greatest kindness.



## So what? How will it change our future?



The Microsoft PROSE team's research serves as a highly precise ruler for global office software developers.



In the past, various vendors mostly stayed in the "passive response" stage when creating "smart spreadsheets"—you input a command like "help me sum this column," and it responds. This non-modal "active prediction" lacked evaluation standards, leaving everyone to grope around in the dark.



Now, **the NAPE framework is completely open-sourced**. It tells everyone:



1.   Don't blindly increase model parameters; instead, fine-tune smaller models on the client side for faster responses and lower power consumption.

2.   Focus on developing an "uncertainty trigger mechanism," allowing AI to assess its own confidence and only present itself to users when it's about 80-90% sure, otherwise remaining an invisible helper.



Perhaps in the near future, when you open Excel again ready to tackle your tasks, you'll find that silent AI has become incredibly intuitive. As soon as you type out two department names, it will have quietly set up the entire structure of your report for you; all you need to do is lightly press the `Tab` key to agree.



This is how technology should be: subtly enhancing our lives while liberating humanity from tedious “hyperactive” operations.





<script> document.addEventListener('DOMContentLoaded', function() {     const content = document.querySelector('.post-content');     if (!content) return;     const headings = content.querySelectorAll('h1, h1');     if (headings.length === 0) return;      
                                                                   const toc = document.createElement('aside');     toc.id = 'dynamic-toc';     toc.innerHTML = '<div class="toc-title">目录</div><ul></ul>';     const ul = toc.querySelector('ul');      headings.forEach((heading, index) => {         if (!heading.id) heading.id = 'toc-head-' + index;         const li = document.createElement('li');         const a = document.createElement('a');         a.href = '#' + heading.id;         a.textContent = heading.textContent;         li.appendChild(a);         ul.appendChild(li);     });     document.body.appendChild(toc);      
                                                                   const style = document.createElement('style');     style.textContent = `         #dynamic-toc { position: fixed; top: 50px; right: 30px; width: 240px; max-height: 70vh; overflow-y: auto; padding: 15px; background: rgba(45, 55, 72, 0.95); border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); font-size: 0.9rem; z-index: 100; font-family: sans-serif; }         #dynamic-toc .toc-title { font-weight: bold; margin-bottom: 10px; color: #00bcd4; border-bottom: 1px solid #4a5568; padding-bottom: 5px; }         #dynamic-toc ul { list-style: none; padding-left: 0; margin: 0; }         #dynamic-toc li { margin: 6px 0; }         #dynamic-toc a { color: #a0aec0; text-decoration: none; display: block; transition: color 0.2s; line-height: 1.4; }         #dynamic-toc a:hover { color: #fff; }         @media screen and (max-width: 1200px) { #dynamic-toc { display: none; } }     `;     document.head.appendChild(style); }); </script>
**结论**：SoftCoT 用一个小型“助手模型”先生成连续空间里的**软思维 token**，再经**投影层**映射到主干 LLM 的表征空间，注入后由**冻结的**主干 LLM完成推理与回答。训练只微调投影层等轻量模块，避免灾难性遗忘；推理时一次前向就得到整段软思维，低开销高增益。([arXiv][1])&#x20;

# 机制

* **三组件**：① 软思维生成模块（小助手模型给出若干“思维向量”）② 投影模块 $W$ 将助手隐状态映到主 LLM 嵌入空间 ③ 主 LLM 的 CoT 解码模块。主 LLM 参数冻结。([arXiv][1])
* **关键做法**：用助手模型最后一层隐状态当作“软 token”，而非离散词；用线性或小 MLP 投影到主模型嵌入，再与任务指令一起喂给主模型继续生成“理由+答案”。([arXiv][1])
* **训练目标**：标准语言建模 NLL，对“理由段+答案段”做监督；遮蔽理由前的部分以防信息泄漏；只调投影层等小块，主干 LLM 冻结。([arXiv][1])
* **推理路径**：问题 → 助手一次前向生成整段软思维 → 投影 → 主 LLM 生成简洁理由与答案。软思维一次性“投喂”，减少逐 token 生成。([arXiv][1])

# 为什么有效

* **连续空间表达更自由**：摆脱离散词表限制，软思维可承载更浓缩的中间信息。([arXiv][1])
* **避免遗忘**：不改主干参数，规避 Coconut、CCoT 等直接微调带来的遗忘风险。([arXiv][1])
* **轻量与可复用**：仅训练投影与少量头部，适配不同主干方便；作为“外接潜在思维”模块可叠加到现有指令模板。

# 与同类对比（潜在/连续式推理）

* **Coconut**：直接在连续空间循环“思维向量”，需微调主干，易遗忘；SoftCoT 冻结主干，用外接软思维。([OpenReview][2], [arXiv][1])
* **CCoT**：通过压缩“沉思 token”提升效率，但仍需改主干；SoftCoT 改为外接+投影。

# 训练与数据

* **监督信号**：带理由标注的数据集上做 LM 训练，监督“理由+答案”。([arXiv][1])
* **基准**：GSM8K、ASDiv、AQuA、StrategyQA、Date Understanding 等五类任务上验证，零样本 CoT 与自一致性均可叠加并带来进一步收益。([arXiv][1])

# 计算与集成

* **成本**：助手一次前向 + 主干常规生成；软思维是“几步向量”，比长 CoT 更省 token。([arXiv][1])
* **可与自一致性结合**：在少样本多路投票场景仍有增益，互补而非替代。([arXiv][1])
* **在综述定位**：被归为“潜在表征压缩/连续空间推理”，通过外接模块增强而不动主干参数。

# 局限

* 软思维**不等于**完整搜索过程，离散生成阶段仍是搜索核心；大模型可扩展性仍待更多实证。([arXiv][1])

# 进阶：SoftCoT++

* **动机**：连续空间的软思维对同一输入通常是**确定的**，缺少采样多样性，不利于测试时扩展。
* **做法**：引入多组“初始软 token”扰动潜在空间，并用对比学习鼓励多样；支持与离散自一致性联合扩展。([arXiv][3])

# 最小落地流程

1. 选主干 LLM（冻结）和小助手模型。
2. 设定软思维长度 $m$ 与投影层 $W$。
3. 训练：$\text{Loss}=\text{NLL}(\text{rationale}+\text{answer})$，仅更新 $W$（或连同极少量头部）。([arXiv][1])
4. 推理：助手→软思维→投影→主干生成理由与答案；可叠加自一致性或 SoftCoT++ 的潜在多样化。([arXiv][1])

> 参考：SoftCoT 原文与增补版，及综述中的定位与描述。([arXiv][1], [ACL Anthology][4])&#x20;

[1]: https://arxiv.org/html/2502.12134v1 "SoftCoT: Soft Chain-of-Thought for Efficient Reasoning with LLMs"
[2]: https://openreview.net/forum?id=tG4SgayTtk&utm_source=chatgpt.com "Training Large Language Model to Reason in a ..."
[3]: https://arxiv.org/abs/2505.11484?utm_source=chatgpt.com "SoftCoT++: Test-Time Scaling with Soft Chain-of-Thought Reasoning"
[4]: https://aclanthology.org/2025.acl-long.1137.pdf?utm_source=chatgpt.com "Soft Chain-of-Thought for Efficient Reasoning with LLMs"

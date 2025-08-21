**定义**：RSD 用“小草稿模型 + 过程奖励模型(PRM)”在**步级**打分，只有当草稿步的奖励低于阈值时才调用“大目标模型”纠正；放宽传统SD的“无偏精确匹配”，以奖励引导的**混合分布**取代硬验证，从而在多步推理中更省算力。([arXiv][1])

# 机制

* **奖励门控**：每步草稿 $y_i$ 得到 PRM 分数 $r(y_i|z_i)$。若 $r\ge \delta$ 则直接接受并续写；否则调用目标模型做“推测纠正”。([arXiv][1])
* **放宽无偏性→混合采样**：不再要求草稿与目标逐 token 精确一致，改为按奖励权重在草稿分布 $P_m$ 与目标分布 $P_M$ 间**自适应混合**（PRSD），并配合拒绝采样以可扩展实现。([arXiv][1])
* **阈值最优性**：在采样预算约束下，**二值阈值**权重最优：

  $$
  \omega_r(y|z)=\mathbb{1}\{r(y|z)\ge \delta_\gamma(z)\},
  $$

  实作可用常数阈值 $\delta$。([arXiv][1])

# 与传统 Speculative Decoding 的差异

* 传统SD：为保证无偏，草稿与目标不匹配即丢弃，实践中还受浮点误差影响，常低于目标模型上限。RSD用PRM保留“高质量不匹配”，整体更稳。([arXiv][1])

# 算法步骤（推理）

1. 草稿模型生成下一步候选；2) PRM打分；3) 若 $r\ge\delta$ 接受并继续用草稿生成；否则调用目标模型对该步进行纠正与续写；4) 重复到终止。([arXiv][1])

# 超参与实践

* **阈值 $\delta$**：经验优选 $0.6\!-\!0.8$，$\delta=0.7$ 跨任务稳健。([arXiv][1])
* **自适应算力分配**：题目越难，越常触发目标模型；易题多由草稿独立完成。([arXiv][1])

# 成效

* 对比仅用目标模型解码：FLOPs 最多**减少 4.4×**。
* 对比并行解码基线：平均**准确率 +3.5**。([arXiv][1])
* 在综述中被归入“**奖励引导的高效推理**”，训练自由，面向多步推理；表4给出定位与对比。&#x20;

# 何时采用

* 长链数理或代码推理。
* 服务侧需在**时延/算力**与**准确率**间做细粒度权衡。([arXiv][1])

> 论文与代码：方法、阈值最优性、消融与基准详见 arXiv 与 ICML 2025 版本。([arXiv][1], [openreview.net][2])

[1]: https://arxiv.org/pdf/2501.19324 "Reward-Guided Speculative Decoding for Efficient LLM Reasoning"
[2]: https://openreview.net/forum?id=AVeskAAETB&noteId=UGuPgWSDLu&utm_source=chatgpt.com "Reward-Guided Speculative Decoding for Efficient LLM ..."

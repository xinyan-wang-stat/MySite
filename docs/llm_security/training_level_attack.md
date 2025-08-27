好的，接着把 **2. Training-level Attacks (Model Poisoning)** 系统梳理。

---

# 2. Training-level Attacks (Model Poisoning)

这些攻击不改模型推理时的输入，而是**在训练/微调阶段注入恶意数据**，让模型带有隐蔽漏洞或后门。

---

## 2.1 Data Poisoning

* **Goal**: 在训练集中埋入恶意样本，改变模型决策边界。
* **Mechanism**:

  * **Label-flipping** → 把一些样本错误标注（猫标成狗）。
  * **Gradient-shifting** → 精心设计的少量中毒样本扭曲参数更新。
  * **Clean-label poisoning** → 看似合理，但语义层面暗藏攻击意图。
* **Impact**:

  * 全局性能下降（accuracy drop）。
  * 在特定输入模式下触发错误输出。
* **Metrics**: ASR (attack success rate), clean accuracy degradation, stealthiness.
* **Defense**: 数据审计与清洗、鲁棒训练（noise-tolerant）、数据溯源、异常梯度检测。

---

## 2.2 Backdoor Attacks

* **Goal**: 植入一个“触发器” (trigger)，在推理时一旦输入出现特定模式，模型输出恶意结果。
* **Mechanism**:

  * 在训练集中加入 (trigger → label) 的样本。
  * 例如：句子里有特殊 token "cf7#" → 模型一定输出“yes”。
* **Impact**:

  * 正常数据时表现无差；
  * 含触发器时 → 攻击者可精确控制输出。
* **Metrics**: Backdoor ASR (带触发器时的攻击成功率)，Clean Accuracy (正常样本表现)。
* **Defense**:

  * 数据集过滤 (trigger pattern detection)。
  * 神经清洗 (neural cleansing) → 找可疑特征。
  * Fine-pruning → 剪掉可能承载后门的神经元。
  * 强化验证（多输入变种测试）。

---

## 2.3 Trojan Models (variant of backdoors)

* **Goal**: 通过共享模型或开源权重传播后门。
* **Mechanism**: 攻击者发布被“注射”过的模型权重，用户直接下载使用。
* **Impact**: 部署方完全不知情，却运行了潜在的后门模型。
* **Defense**:

  * 权重来源认证（签名/哈希校验）。
  * 行为一致性测试。
  * 模型反向工程检查。

---

## 2.4 Training-time Evasion & Poisoning in RLHF / Fine-tuning

* **Goal**: 在对齐阶段引导模型学习错误偏好。
* **Mechanism**:

  * **Preference poisoning**：标注者或对抗者在 RLHF 偏好数据中插入恶意排序。
  * **Instruction-tuning poisoning**：在指令微调语料里混入恶意 prompt-response 对。
* **Impact**:

  * 模型在某些任务上出现系统性偏差。
  * 可被利用为长期后门。
* **Defense**: 标注来源管控、多审员一致性、对抗性验证集。

---

# 3. Evaluation Metrics for Training-level Attacks

* **Clean accuracy** → 模型在正常数据上的表现。
* **Attack success rate (ASR)** → 特定条件下是否被触发。
* **Stealthiness** → 攻击样本是否可被检测到。
* **Transferability** → 在不同任务/数据上是否仍能触发。

---

# 4. Defensive Baselines

* **Data pipeline security**：数据溯源、供应链完整性。
* **Robust training**：对抗训练、差分隐私、噪声注入。
* **Post-hoc detection**：神经清洗、激活聚类、异常值检测。
* **Model validation**：红队测试，触发器扫描，watermark 对比。

---

📌 总结：
Training-level Attacks 就是 **在模型训练/微调时种下“毒种”或“后门”**。推理时看起来正常，但在攻击者控制条件下，模型会泄露、偏差或直接被劫持。

---

要不要我接着讲 **3. Model-level Attacks (参数与隐私层面)**？

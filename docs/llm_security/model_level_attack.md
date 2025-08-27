好的，下面系统梳理 **3. Model-level Attacks**。

---

# 3. Model-level Attacks (Direct Exploitation of Parameters)

这类攻击不依赖输入投毒或训练数据，而是直接针对模型本身的**参数、内部机制、隐私信息**。

---

## 3.1 Model Extraction / Stealing

* **Goal**: 攻击者通过查询模型 API，训练出一个近似的复制模型。
* **Mechanism**:

  * Query-based distillation：大量输入–输出对收集。
  * 用蒸馏或模仿学习训练本地模型。
* **Impact**: 知识产权盗窃、绕过付费 API、转移安全弱点。
* **Metrics**: Fidelity (复制模型与原模型输出一致性)、功能覆盖度。
* **Defense**:

  * API rate limiting / 水印 (watermarking) 输出。
  * Query auditing（检测大规模可疑调用）。
  * Output perturbation（对抗蒸馏的细噪声）。

---

## 3.2 Membership Inference Attacks (MIA)

* **Goal**: 判断某个样本是否在模型训练集里。
* **Mechanism**:

  * 比较模型对输入的置信度 / 过拟合特征。
  * Shadow model 攻击：训练多个 shadow model 来估计差异。
* **Impact**: 训练数据隐私泄露（可能包含敏感信息）。
* **Metrics**: Membership inference accuracy (高于随机猜测)。
* **Defense**:

  * Differential privacy training (DP-SGD)。
  * Regularization 减少过拟合。
  * Confidence masking / 输出限制。

---

## 3.3 Model Inversion

* **Goal**: 从模型预测中反推训练样本的特征。
* **Mechanism**:

  * 给定某类标签，迭代生成输入，使模型预测该标签概率最大。
  * 最终得到近似于训练样本的“反演图像/文本”。
* **Impact**: 个人隐私数据泄露（如人脸、医疗记录）。
* **Defense**:

  * Differential privacy。
  * 限制输出置信度与中间表示。
  * 模型蒸馏降低信息泄露。

---

## 3.4 Fine-tuning-based Backdoor Injection

* **Goal**: 通过用户可访问的微调接口，往现有模型参数里注入后门。
* **Mechanism**:

  * 利用开放 fine-tuning API 提交恶意数据。
  * 在基础模型中埋入触发器模式。
* **Impact**: 模型在正常任务正常，但触发条件下被劫持。
* **Defense**: 微调数据审计、沙箱化训练环境、输出检测。

---

## 3.5 Side-channel & Gradient Leakage

* **Goal**: 从训练过程或推理 API 的副信道泄露信息。
* **Mechanism**:

  * 训练梯度上传（分布式/联邦学习场景） → 重建原始输入。
  * Timing / memory access 模式。
* **Impact**: 原始训练数据泄露。
* **Defense**:

  * Secure aggregation in federated learning。
  * Gradient clipping + DP。
  * Homomorphic encryption / secure enclave。

---

# Metrics for Model-level Attacks

* Extraction fidelity (近似程度)。
* Membership inference accuracy。
* Inversion reconstruction quality (PSNR, BLEU)。
* Attack stealthiness。

---

# Defensive Baselines

* **Differential privacy**：全局约束训练时的隐私泄漏。
* **Regularization**：减少过拟合，降低 MIA 成功率。
* **Access control**：限制 fine-tuning / gradient 接口。
* **Output sanitization**：避免过高置信度。
* **Watermarking**：标记输出以追踪滥用。

---

📌 总结：
Model-level Attacks 本质上是“从模型本身榨干秘密”或“在模型内部埋入漏洞”。包括 **参数窃取、训练集泄露、反演隐私、后门注入、梯度泄漏**。

---

要不要我继续梳理 **4. System-level Attacks (Pipeline & Deployment)**？

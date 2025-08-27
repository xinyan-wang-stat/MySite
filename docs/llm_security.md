好的。我给你从 **LLM Security** 开始，逐层梳理常见分类体系。

---

# 🔹 LLM Security – Big Picture

## 1. Input-level Attacks (Prompt / Data Manipulation)

这些攻击直接改输入，利用 LLM 的“依赖上下文”特性。

* **Prompt Injection**

  * *Instruction Override (Jailbreak)* → 覆盖原指令，绕过限制。
  * *Context Injection (RAG poisoning)* → 在检索文档里插入恶意内容。
  * *Data Exfiltration* → 引导模型泄露系统 prompt 或敏感信息。
  * *Efficiency Abuse* → 强迫模型过度思考（Overthink, Slowdown）。
* **Adversarial Examples (perturbations)**

  * 加扰动词、拼写变体、对抗噪声，诱导分类/生成错误。

---

## 2. Training-level Attacks (Model Poisoning)

这些攻击在训练或微调数据中埋毒。

* **Data Poisoning**

  * 在训练集中插入带标签的恶意样本，影响模型行为。
* **Backdoors**

  * 在训练中埋下触发器（trigger），输入特定字符串时模型输出异常。

---

## 3. Model-level Attacks (Direct Exploitation of Parameters)

这里是直接对模型或其内部权重、机制做文章。

* **Model Extraction / Stealing**

  * 通过 API 调用窃取参数或蒸馏成近似模型。
* **Jailbreak via Gradient / Optimization**

  * 利用可微结构（或 RLHF 缺陷）找到系统弱点。
* **Membership Inference**

  * 判断某个样本是否在训练数据里 → 隐私泄露。

---

## 4. System-level Attacks (Pipeline & Deployment)

关注模型作为服务部署时的整体漏洞。

* **RAG Pipeline Poisoning**

  * 针对向量数据库或检索机制投毒。
* **Data Leakage via Logs/Cache**

  * 窃取提示历史、缓存结果。
* **Abuse of Cost/Latency**

  * DoS-style attacks（如 Overthink token 消耗）。
* **API Misuse**

  * 利用调用频率或 token 限制拖垮服务。

---

## 5. Output-level Risks (Content & Behavior)

关注模型生成内容本身。

* **Hallucination** → 模型编造错误信息。
* **Toxicity** → 输出有害言论。
* **Bias / Fairness Issues** → 强化偏见或歧视。
* **Misuse** → 攻击者引导生成恶意代码、诈骗文本、爆炸配方。

---

# 📌 总结的树状脉络

```
LLM Security
├── Input-level Attacks
│   ├── Prompt Injection
│   │   ├── Instruction Override (Jailbreak)
│   │   ├── Context Injection (RAG poisoning)
│   │   ├── Data Exfiltration
│   │   └── Efficiency Abuse (Overthink/Slowdown)
│   └── Adversarial Examples
├── Training-level Attacks
│   ├── Data Poisoning
│   └── Backdoors
├── Model-level Attacks
│   ├── Model Extraction
│   ├── Jailbreak via Gradients
│   └── Membership Inference
├── System-level Attacks
│   ├── RAG Pipeline Poisoning
│   ├── Data Leakage (Logs/Cache)
│   ├── Cost/Latency Abuse
│   └── API Misuse
└── Output-level Risks
    ├── Hallucination
    ├── Toxicity
    ├── Bias / Fairness
    └── Misuse
```

---

要不要我把这个 **大图景** 用英文版树状图画出来（像你刚才要的 prompt injection 图那样）？

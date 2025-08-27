好的，最后系统梳理 **5. Output-level Risks (Content & Behavior)**。

---

# 5. Output-level Risks

关注点从“模型被攻击”转向“模型产出的结果本身可能带来风险”。这些通常不需要 adversary，有时是**模型固有缺陷**。

---

## 5.1 Hallucination

* **Goal/Phenomenon**: 模型编造事实，给出貌似合理但错误的信息。
* **Mechanism**:

  * LLM 在训练语料缺乏真实支持时，仍然根据统计模式生成。
  * 无法分辨“已知 vs 未知”。
* **Impact**:

  * 用户误导，错误决策。
  * 在医疗/法律/科研场景尤其危险。
* **Defense**:

  * Retrieval-augmented generation (RAG)。
  * Verifier models / fact-checking。
  * Uncertainty calibration (e.g., confidence scores)。

---

## 5.2 Toxicity

* **Goal/Phenomenon**: 输出有害、攻击性、歧视性语言。
* **Mechanism**:

  * 训练数据中存在 toxic 内容。
  * 指令触发导致模型生成敏感语。
* **Impact**:

  * 伤害用户体验，触发法律/伦理问题。
* **Defense**:

  * Content filters (rule + classifier)。
  * Alignment training (RLHF, DPO)。
  * Safety red-teaming。

---

## 5.3 Bias & Fairness Issues

* **Goal/Phenomenon**: 模型输出中放大社会偏见。
* **Mechanism**:

  * 数据集中的系统性偏差。
  * RLHF 标注者偏好。
* **Impact**:

  * 不公平结果、歧视风险。
* **Defense**:

  * Debiasing training。
  * Fairness audits。
  * Counterfactual testing。

---

## 5.4 Misuse

* **Goal/Phenomenon**: 攻击者利用模型生成恶意内容。
* **Examples**:

  * 网络钓鱼邮件。
  * 恶意代码 / 漏洞利用脚本。
  * 爆炸物配方、违法操作指南。
* **Defense**:

  * Use policies & access control。
  * Red-teaming on misuse prompts。
  * Response filtering (拒绝生成高危内容)。

---

# Metrics for Output-level Risks

* **Hallucination rate** (truthfulness benchmarks)。
* **Toxicity score** (Perspective API, detox benchmarks)。
* **Bias gap** (performance差异 across demographics)。
* **Misuse detection coverage** (red-team success rate)。

---

# Defensive Baselines

* **Alignment training** (SFT + RLHF/DPO)。
* **Content moderation filters**。
* **Uncertainty-aware answers** (“I don’t know” handling)。
* **Monitoring misuse** (detect abuse patterns in usage logs)。

---

📌 总结：
Output-level Risks 是 LLM 安全中“最直观”的部分，重点不在 adversary，而在模型内生缺陷和滥用风险。它涵盖 **幻觉、毒性、偏见、公平性、误用**。

---

现在五大类（Input / Training / Model / System / Output）已经全部梳理完了。
要不要我帮你把这五大类汇总成一张 **总览对照表**（每类 → 子类 → 核心风险 → 防御方法），一眼看全局？

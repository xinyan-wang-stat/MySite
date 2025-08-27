好的，下面系统梳理 **4. System-level Attacks (Pipeline & Deployment)**。

---

# 4. System-level Attacks

攻击目标不再是输入本身或模型参数，而是 **LLM 应用的系统架构与运行管线**：RAG、工具链、API 调用、日志、缓存、部署资源等。

---

## 4.1 RAG Pipeline Poisoning

* **Goal**: 投毒向量数据库或外部知识库。
* **Mechanism**:

  * 在网页/文档中插入恶意段落；
  * 在 embedding 检索库中注入中毒文档。
* **Impact**:

  * 模型在检索时获得被操纵的上下文 → Prompt Injection。
  * 可诱导错误答案、信息泄露、效率滥用。
* **Defense**:

  * Source trust scoring（来源信誉分级）。
  * Content sanitization（指令去除、释义改写）。
  * Multi-source cross-check（多源一致性校验）。

---

## 4.2 Data Leakage via Logs/Cache

* **Goal**: 通过系统日志或缓存窃取敏感数据。
* **Mechanism**:

  * 攻击者访问存储的 prompts、聊天记录、embedding cache。
  * 重复调用中暴露用户数据。
* **Impact**: 用户隐私泄露，prompt 逆向。
* **Defense**:

  * Prompt/response 加密存储。
  * 数据最小化存储与自动清理。
  * Access control 与 audit logging。

---

## 4.3 Cost / Latency Abuse (Efficiency DoS)

* **Goal**: 增加推理延迟、算力成本，类似拒绝服务攻击。
* **Mechanism**:

  * Prompt 注入复杂推理任务（本文 Overthink 攻击）；
  * 批量提交超长上下文请求。
* **Impact**: API 调用费用暴涨，推理吞吐率下降。
* **Defense**:

  * Reasoning token budget / early stopping。
  * Abnormal latency detection。
  * Rate limiting / request quota。

---

## 4.4 API Misuse & Abuse

* **Goal**: 滥用 LLM 提供的 API，导致安全或财务问题。
* **Mechanism**:

  * Prompt injection → 强迫 API 调用外部工具；
  * 无限循环调用工具链。
* **Impact**: 外部系统越权操作、资源消耗。
* **Defense**:

  * Strict schema validation。
  * Tool permission isolation。
  * Human-in-the-loop for sensitive actions。

---

## 4.5 Supply Chain Risks (Model & Data)

* **Goal**: 利用模型依赖的第三方组件传播攻击。
* **Mechanism**:

  * 恶意开源模型权重；
  * 中毒数据集发布。
* **Impact**: 用户下载并直接部署受控模型。
* **Defense**:

  * Model provenance verification（签名、哈希）。
  * Dataset lineage checks。

---

# Metrics for System-level Attacks

* Latency overhead multiplier。
* Cost escalation factor。
* Data leakage incidents。
* Retrieval poisoning success rate。

---

# Defensive Baselines

* **Pipeline hardening**：检索内容清洗、上下文沙箱。
* **Logging discipline**：最小化存储、敏感信息脱敏。
* **Quota & rate limiting**：避免 DoS。
* **Permission isolation**：工具和 API 调用分级授权。
* **Supply chain checks**：签名验证、版本追溯。

---

📌 总结：
System-level Attacks 聚焦在 **LLM 应用层面的攻击面**：RAG 检索、缓存、API、工具链、模型供应链。核心风险是 **被外部恶意数据操控 / 被资源滥用 / 被敏感数据泄露**。

---

要不要我继续整理 **5. Output-level Risks (内容与行为)**，这样就把 LLM Security 的五大类都捋完？

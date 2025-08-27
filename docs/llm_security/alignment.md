好，我们展开 **(c) 对齐 (Alignment)**，重点放在 **RLHF、DPO、GRPO**。

---

## 1. 对齐 (Alignment) 的目标

* 预训练 / 微调后的 LLM 虽然会生成语言，但可能：

  * 不符合人类偏好（答非所问、不安全）。
  * 输出啰嗦、不简洁。
  * 在模糊问题上没法给合理答案。
* **对齐 (alignment)** 的任务 = 让模型更符合人类期望。
* 常见做法：用 **人类反馈** 或 **人工标注的偏好数据** 来调整模型。

---

## 2. RLHF (Reinforcement Learning with Human Feedback)
结论：**RLHF = SFT → 奖励模型(RM) → RL(通常PPO) + KL 约束**。用人类偏好把策略往“更合人意”的区域推，同时用 KL 把策略钉在参考策略附近，防崩。

---

# 一、三步管线

## 1) SFT（监督微调，起点策略）

在指令数据上做交叉熵，得参考策略 $\pi_{\text{ref}}=\pi_{\text{SFT}}$。

$$
\min_\theta \;-\!\sum_t \log \pi_\theta(y_t|x, y_{<t})
$$

## 2) 奖励模型 RM（把偏好变成分数）

数据是偏好对 $(x,y^+,y^-)$。训练标量打分 $r_\phi(x,y)$，用 Bradley–Terry/对比损失：

$$
\min_\phi \;-\log \sigma\!\big(r_\phi(x,y^+)-r_\phi(x,y^-)\big)
$$

## 3) 强化学习（PPO 最常见）

把“写一段回复”当一回合：状态=已生成前缀，动作=下一个 token，episode 回报在**末尾**给出：

$$
R(x,y)=r_\phi(x,y) \;-\; \beta\,\mathrm{KL}\!\big(\pi_\theta(\cdot|x)\,\|\,\pi_{\text{ref}}(\cdot|x)\big)\;-\;\lambda_{\text{len}}\cdot \text{len}(y)
$$

* 第一项：人类偏好方向
* 第二项：**KL 正则**，防偏离 SFT
* 第三项：长度罚，防话痨

PPO 更新（分词级优势 $A_t$ 用 GAE 估计）：

$$
\max_\theta \;\mathbb{E}\!\left[\min\!\big(\rho_t A_t,\;\mathrm{clip}(\rho_t,1\!\pm\!\epsilon)A_t\big)\right]
\;-\;\lambda_{KL}\,\mathrm{KL}_t
$$

$$
\rho_t=\exp\big(\log\pi_\theta(a_t|s_t)-\log\pi_{\theta_{old}}(a_t|s_t)\big)
$$

> 备注：KL 可“显式项”加在目标里，或“内嵌”到回报里（二选一，工程上常用**目标 KL**跟踪并自适应调 $\beta$)。

---

# 二、直观机制

* **RM**学“人更喜欢哪种回答”。
* **PPO**把概率质量从差回答移到好回答上。
* **KL**像“松紧带”，限制每步更新幅度，稳定训练。

---

# 三、最小实现流程（伪代码）

```text
# 预备：已有 pi_ref (SFT)、RM
repeat:
  batch X ~ prompts
  # 采样策略：从当前 pi_theta 生成 K 个候选
  Y = sample(pi_theta, X, num_samples=K, temperature=0.7)
  # 评分：端到端奖励
  R = RM.score(X, Y) - beta*KL(pi_theta||pi_ref) - lambda_len*len(Y)
  # 展开成逐token轨迹，估计 advantage (GAE)
  A_t, V_t = estimate_advantage(R, trajectories)
  # PPO 更新
  theta <- argmax L_clip(theta; A_t, rho_t) - lambda_KL*KL_t + entropy_bonus
until dev_metrics converge
```

---

# 四、关键超参与监控

* **目标 KL（nats/token）**：常设目标值并自调 $\beta$。目标太小→学不到偏好；太大→漂离 SFT。
* **采样 K**：每条 prompt 采样 1–4 条，提高探索但控制算力。
* **长度/重复惩罚**：必要，防“冗长”“复读”。
* **监控**：偏好命中率↑、离线 RM 分数↑、目标 KL 贴合、人工验收集稳定。

---

# 五、常见陷阱

* **奖励黑客**：模型学会讨好 RM 的“捷径”。解法：多源偏好、对抗负样本、校准 RM、增加格式/事实校验。
* **话痨**：没有长度/重复罚则，RM 往往偏长。
* **模式坍塌**：KL 太弱或更新太猛。降学习率，升 $\beta$/目标 KL。
* **RM 偏置**：单一域训练导致泛化差。混合域+人工抽检。
* **稀疏回报**：只在句尾给分不稳定。可加**中间过程分**（PRM）或 step 级 shaping。

---

# 六、与 DPO 的对照

* **RLHF**：显式 RM + RL（PPO），可接第三方工具回报，灵活但工程复杂。
* **DPO**：直接在偏好对上做对比优化，无 RM、无 RL，稳定简单。
* **取舍**：需要在线探索/工具复合回报 → RLHF；纯偏好拟合/工程简洁 → DPO。

---

# 七、极简“看懂一眼”的数学骨架

* 偏好到分数：$\;y^+>y^- \Rightarrow r_\phi(x,y^+)>r_\phi(x,y^-)$
* 回报成形：$\;R = r_\phi - \beta\,\mathrm{KL} - \lambda_{\text{len}}\text{len}$
* 策略更新：$\;\theta \leftarrow \theta + \eta \,\nabla_\theta \mathbb{E}\big[\sum_t \log\pi_\theta(a_t|s_t) A_t\big]$（PPO 用裁剪与 KL 稳定化）

---

需要我把“**如何设目标 KL 与自适应 $\beta$**”“**用 PRM 做过程回报**”这两点再展开成可直接落地的配置与公式吗？


# PPO
好的，我直接给你写一遍 **PPO 的 Actor–Critic 更新流程**，在原始 Actor–Critic 的框架里插入 PPO 的修改。

---

## PPO 更新流程（基于 Actor–Critic）

### 输入

* Actor（策略网络）参数 $\theta$，输出 $\pi_\theta(a|s)$
* Critic（价值网络）参数 $\phi$，输出 $V_\phi(s)$
* 环境交互数据 $(s_t,a_t,r_t,s_{t+1})$

---

### Step 1. 采样

用当前策略 $\pi_{\theta_{\text{old}}}$ 跑一批 trajectory，得到经验 $\{s_t,a_t,r_t\}$。

---

### Step 2. Critic 更新

* TD 目标：

  $$
  \hat{G}_t = r_t + \gamma V_\phi(s_{t+1})
  $$
* Critic 损失：

  $$
  L^{\text{VF}}(\phi) = (V_\phi(s_t) - \hat{G}_t)^2
  $$
* 更新 Critic 参数 $\phi \leftarrow \phi - \beta \nabla_\phi L^{\text{VF}}(\phi)$。

---

### Step 3. 优势函数估计

* TD 误差：

  $$
  \delta_t = r_t + \gamma V_\phi(s_{t+1}) - V_\phi(s_t)
  $$
* 用 GAE 累积：

  $$
  A_t = \sum_{l=0}^\infty (\gamma \lambda)^l \delta_{t+l}
  $$

---

### Step 4. Actor 更新（PPO 替换原始 PG）

* 概率比率：

  $$
  r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{\text{old}}}(a_t|s_t)}
  $$
* PPO Actor 损失：

  $$
  L^{\text{CLIP}}(\theta) = \mathbb{E}_t\Big[\min(r_t(\theta)A_t,\;\text{clip}(r_t(\theta),1-\epsilon,1+\epsilon)A_t)\Big]
  $$
* 更新 Actor 参数：

  $$
  \theta \leftarrow \theta + \alpha \nabla_\theta L^{\text{CLIP}}(\theta)
  $$

---

### Step 5. 重复

* 多个 mini-batch，迭代 $K$ 次更新 Actor 和 Critic。
* 更新完成后，把 $\theta$ 存为新的 $\theta_{\text{old}}$，进入下一轮采样。

---

## 总结版

**Actor–Critic 原流程：**
采样 → 更新 Critic（MSE） → 计算优势 → 更新 Actor（policy gradient）

**PPO 替换版：**
采样 → 更新 Critic（MSE） → 计算优势 → 更新 Actor（用 $L^{\text{CLIP}}$ 而不是简单的 $\log\pi \cdot A$）

---

要不要我把这套流程翻译成 **Python 伪代码**，像一段最小可运行的训练循环？


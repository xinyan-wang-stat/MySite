结论：Token‑Budget=**为每道题找到最小可行预算 $B^\*$**，用这个预算生成“短且对”的CoT，再做SFT固化这种风格。下面给出工程级细化。

# 1. 预算定义与成功判定

* **预算类型**：优先用“思考token预算”对 `<think>…</think>` 计数；也可用“总token预算”。
* **成功判定**：答案正确且格式合规（必须分离 `<think>/<answer>`），否则视为失败。
* **单题目标**：

  $$
  B^\*(x)=\min\{B:\ \text{Solve}(x,\ B)\ \text{成功}\}
  $$

# 2. 二分搜索 $B^\*$（稳健版）

* 区间初始化：

  * $B_\min$：如 16；$B_\max$：如 512/768（按模型与任务设上限）。
  * 若首次在 $B_\max$ 都失败，标记为**难例**，走备用通道（见 §6）。
* 单次评估：在预算 $B$ 下生成 $m$ 次（如 $m=2\!-\!3$），若**任一**通过即视为成功（降噪）。
* 迭代：成功→收缩上界，失败→抬高下界；重复至区间宽度 ≤ δ（如 8 token）。
* 复杂度：$O(\log(B_\max\!-\!B_\min)\times m)$。

# 3. 生成与数据落库

* 用 $B^\*(x)$ **再生成一次**短链并落库：存 `(x, short‑CoT, answer, B*)`。
* 质检：答案一致；`<think>` 内不得出现“省略”“跳过计算”之类投机文本；压缩率 ≥ 阈值（如 ≥40%）。

# 4. SFT 训练配方

* 目标：交叉熵模仿短链；主数据=Token‑Budget 样本；**混入10–20%原始长链**稳住难题泛化。
* 课程学习（可选）：先用较小 $B$ 样本训练，逐步加入较大 $B$ 样本，避免一次性退化。
* 正则：可对 `<think>` 长度加 L2 或 label‑smoothing，防止回涨。

# 5. 推理提示模板（可直接用）

* **硬预算**：

  ```
  <sys>Return reasoning in <think> and final answer in <answer>.</sys>
  <user>
  You must finish your reasoning within at most B=<B> tokens inside <think>.
  Keep it concise and essential. Problem: {QUESTION}
  </user>
  ```
* **软预算 + 风格约束**：

  ```
  Keep reasoning within ~<B> tokens. Avoid repetition, keep only decisive steps.
  ```

# 6. 难例与保护

* **难例回退**：若 $B^\*$ 找不到，标记难例，改用较大固定预算 $B_{\text{large}}$ 生成；或路由到“深推理通道”（如 BoN/Verifier）。
* **最短下限**：为数学/代码设 `B_floor`（如 32/64）防“过压缩直答”。
* **温度与采样**：搜索期用较低温度（如 0.4–0.7）降方差；落库时可用 0.7 提高可读性。

# 7. 评测与可视化

* **曲线一**：固定预算下准确率 $Acc(B)$。
* **曲线二**：在同等准确率下的平均 `<think>` token。
* **面积指标**：$\int Acc(B)\,d\log B$ 近似衡量“效率‑精度”折中。
* **过程/结果效率**：报告每题步骤冗余度、是否存在“再扩展能改答案”。

# 8. 常见坑

* **误把 `<answer>` token 计入预算** → 训练时答复异常简短。
* **单次判定噪声大** → 每点多次采样取“任一成功”为成功。
* **过拟合短链风格** → 混入少量长链 + 难例回退。
* **提示被忽略** → 在系统提示中**强制** `<think>/<answer>` 结构，并在格式不合规时判失败。

# 9. 与相邻方法协同

* **Self‑Training**：先做 Token‑Budget 得到“最短可行”样本，再在同题多样采样中选更佳文风的短链充实数据。
* **C3oT/TokenSkip**：对 $B^\*$ 样本再做压缩，进一步降冗。
* **SR/DPTS**：在线阶段用奖励或置信度做**早停/剪枝**，Token‑Budget 负责**离线数据侧**的“最小可行链”。

# 10. 快速默认超参

* 搜索：$B_\min=16$, $B_\max=512$, δ=8, $m=2$, 温度=0.5。
* 训练：batch 128，lr 1e‑5，混入长链 15%，epoch 1–2。
* 质检：压缩率≥40%，答案一致性必须通过，格式严格。

# 11. 迷你伪代码

```python
def find_budget(x, Bmin=16, Bmax=512, delta=8, m=2):
    lo, hi = Bmin, Bmax
    while hi - lo > delta:
        mid = (lo + hi) // 2
        ok = any(solve(x, B=mid, temp=0.5).is_correct() for _ in range(m))
        if ok: hi = mid
        else:  lo = mid + 1
    return hi  # B*

B_star = find_budget(x)
short = solve(x, B=B_star, temp=0.7)      # 生成短且对的链
if qc(short): write_dataset(x, short, B_star)

# SFT: 以短链为主，混少量长链做稳定器
```

要点一句话：**先精确“找预算”，再“按预算生成”，最后“用SFT固化”**。这比只选最短样本更可控，也比纯RL更省心，部署快、收益稳。

---
title: "Mathematical Theory of Adaptive Vocabulary Testing / 自适应词汇测试的数学理论"
author: Zheng Huang
date: 2026-04-06
category: Jekyll
layout: post
---

## 1. Problem Statement / 问题描述

We want to estimate a learner's English vocabulary size $V$ from a limited number of test questions. The learner sees English words of varying difficulty (ranked by corpus frequency) and reports whether they know each word.

我们希望通过有限数量的测试题来估计学习者的英语词汇量 $V$。学习者看到按语料库词频排列的不同难度的英文单词，并报告自己是否认识每个单词。

The core assumption is the **monotone knowledge hypothesis**: a learner who knows a word at frequency rank $r$ is very likely to know all words with rank $< r$ (i.e., more common words). This assumption is well-supported by corpus linguistics research and means there exists a **boundary rank** $\theta$ where knowledge transitions from "known" to "unknown."

核心假设是**单调知识假设**：如果一个学习者认识频率排名为 $r$ 的单词，那么他/她大概率也认识所有排名 $< r$（即更常见的）的单词。这一假设在语料库语言学研究中得到了充分验证，意味着存在一个**边界排名** $\theta$，在此处学习者的词汇知识从"认识"过渡到"不认识"。

---

## 2. Logistic Model / Logistic 模型

We model the probability that a learner knows a word at rank $r$ using the logistic function:

我们使用 logistic 函数来建模学习者认识排名为 $r$ 的单词的概率：

$$
P(\text{know} \mid r) = \frac{1}{1 + e^{\beta(r - \theta)}}
$$

where:

其中：

- $\theta$ = the **vocabulary boundary** (rank where $P = 0.5$). This is our target estimate — the learner's vocabulary size. / **词汇边界**（$P = 0.5$ 处的排名），即我们要估计的词汇量。
- $\beta$ = the **steepness parameter**, controlling how sharply knowledge drops off around $\theta$. A larger $\beta$ means a more abrupt transition. / **陡度参数**，控制知识在 $\theta$ 附近下降的速率。$\beta$ 越大，过渡越陡峭。

When $r \ll \theta$, $P \approx 1$ (the learner almost certainly knows the word). When $r \gg \theta$, $P \approx 0$. At $r = \theta$, $P = 0.5$ exactly.

当 $r \ll \theta$ 时，$P \approx 1$（学习者几乎肯定认识）。当 $r \gg \theta$ 时，$P \approx 0$。在 $r = \theta$ 处，$P = 0.5$。

---

## 3. Maximum Likelihood Estimation / 最大似然估计

Given $n$ test words with observed responses, we estimate $\theta$ and $\beta$ by maximizing the log-likelihood:

给定 $n$ 个测试单词的观察结果，我们通过最大化对数似然函数来估计 $\theta$ 和 $\beta$：

$$
\ell(\theta, \beta) = \sum_{i=1}^{n} \left[ y_i \ln P_i + (1 - y_i) \ln(1 - P_i) \right]
$$

where $y_i \in \{0, 0.5, 1\}$ is the learner's response to word $i$ (know / not sure / don't know), and $P_i = P(\text{know} \mid r_i)$ is the model's predicted probability.

其中 $y_i \in \{0, 0.5, 1\}$ 是学习者对第 $i$ 个单词的回答（认识 / 不确定 / 不认识），$P_i = P(\text{know} \mid r_i)$ 是模型的预测概率。

In practice, we use **stratified band scoring**: the 17,634 words are divided into 50 frequency bands (~353 words each), and we aggregate responses within each band to get a band-level knowledge rate $\bar{y}_b$ with sample size $n_b$. The log-likelihood becomes:

实际操作中，我们使用**分层带评分**：将 17,634 个单词分为 50 个频率带（每带约 353 个单词），并在每个带内汇总回答以获得带级别的知识率 $\bar{y}_b$ 和样本量 $n_b$。对数似然变为：

$$
\ell(\theta, \beta) = \sum_{b=1}^{B} n_b \left[ \bar{y}_b \ln P_b + (1 - \bar{y}_b) \ln(1 - P_b) \right]
$$

where $P_b = \frac{1}{1 + e^{\beta(\bar{r}_b - \theta)}}$ and $\bar{r}_b$ is the midpoint rank of band $b$.

其中 $P_b = \frac{1}{1 + e^{\beta(\bar{r}_b - \theta)}}$，$\bar{r}_b$ 是第 $b$ 个频率带的中点排名。

The MLE is found numerically via grid search followed by fine refinement.

通过网格搜索加精细优化来数值求解 MLE。

---

## 4. Fisher Information and Confidence Intervals / Fisher 信息量与置信区间

The precision of our estimate comes from **Fisher Information**. For the logistic model, the Fisher Information for $\theta$ from a single observation at rank $r$ is:

估计的精度来自 **Fisher 信息量**。对于 logistic 模型，在排名 $r$ 处的单个观测对 $\theta$ 的 Fisher 信息量为：

$$
I_r(\theta) = \beta^2 \cdot P(r) \cdot [1 - P(r)]
$$

This quantity is **maximized** when $P(r) = 0.5$ (i.e., when the test word's difficulty exactly matches the learner's level), giving:

此量在 $P(r) = 0.5$ 时**取最大值**（即测试单词的难度恰好匹配学习者水平），此时：

$$
I_{\max} = \frac{\beta^2}{4}
$$

The total Fisher Information from all test questions is:

所有测试题目的总 Fisher 信息量为：

$$
I_{\text{total}}(\theta) = \sum_{i=1}^{n} \beta^2 \cdot P_i \cdot (1 - P_i)
$$

Or equivalently using band-aggregated data:

或等价地使用带汇总数据：

$$
I_{\text{total}}(\theta) = \sum_{b=1}^{B} n_b \cdot \beta^2 \cdot P_b \cdot (1 - P_b)
$$

The **standard error** of the MLE is:

MLE 的**标准误差**为：

$$
\text{SE}(\hat{\theta}) = \frac{1}{\sqrt{I_{\text{total}}(\theta)}}
$$

And the **95% confidence interval** is:

**95% 置信区间**为：

$$
\hat{\theta} \pm 1.96 \cdot \text{SE}(\hat{\theta}) = \hat{\theta} \pm \frac{1.96}{\sqrt{I_{\text{total}}(\theta)}}
$$

---

## 5. Why Adaptive Testing Narrows the CI / 为什么自适应测试能缩小置信区间

The key insight is that **not all questions are equally informative**. A question at rank $r$ contributes $\beta^2 P(1-P)$ to the total information. Consider two extremes:

关键洞察是**并非所有问题的信息量相等**。排名 $r$ 处的问题贡献 $\beta^2 P(1-P)$ 的信息量。考虑两个极端：

| Scenario / 场景 | $P(\text{know})$ | Information $I$ / 信息量 | Relative efficiency / 相对效率 |
|:---:|:---:|:---:|:---:|
| Word at boundary / 边界处 | 0.5 | $\beta^2/4$ | 100% |
| Easy word / 简单单词 | 0.95 | $0.0475\beta^2$ | 19% |
| Hard word / 难单词 | 0.05 | $0.0475\beta^2$ | 19% |

A question where you almost certainly know (or don't know) the answer contributes roughly **5 times less information** than one at your boundary.

一个你几乎肯定认识（或不认识）的问题，其信息量大约只有边界处问题的 **五分之一**。

This is why the **advanced test** is so much more efficient: by concentrating all questions in the boundary zone, every question contributes near-maximum information.

这就是**高级测试**更高效的原因：通过将所有问题集中在边界区域，每个问题都贡献接近最大的信息量。

---

## 6. CI Width Scaling Law / 置信区间宽度的缩放定律

For optimally targeted questions (all at $P = 0.5$):

对于最优定位的问题（全部在 $P = 0.5$ 处）：

$$
\text{SE}(\hat{\theta}) = \frac{1}{\sqrt{n \cdot \beta^2 / 4}} = \frac{2}{\beta \sqrt{n}}
$$

So the 95% CI half-width is:

因此 95% 置信区间的半宽为：

$$
w = \frac{1.96 \times 2}{\beta \sqrt{n}} = \frac{3.92}{\beta \sqrt{n}}
$$

The critical scaling relationship is:

关键的缩放关系是：

$$
\boxed{w \propto \frac{1}{\sqrt{n}}}
$$

This means:

这意味着：

| To reduce CI width by / 将 CI 宽度缩小 | Need to multiply questions by / 需要将问题数乘以 |
|:---:|:---:|
| 50% (half) | 4× |
| 67% (one-third) | 9× |
| 75% (one-quarter) | 16× |

However, this assumes all questions are optimally placed. In the **quick test**, many questions are spent scanning non-boundary regions. If only a fraction $f$ of questions are near the boundary, the effective sample size is roughly $n_{\text{eff}} \approx f \cdot n$. The advanced test achieves $f \approx 1$, versus $f \approx 0.3$ for the quick test.

但这假设所有问题都是最优放置的。在**快速测试**中，许多问题花费在扫描非边界区域。如果只有比例 $f$ 的问题在边界附近，有效样本量大约为 $n_{\text{eff}} \approx f \cdot n$。高级测试达到 $f \approx 1$，而快速测试约为 $f \approx 0.3$。

---

## 7. Numerical Example / 数值示例

Suppose after the quick test: $\hat{\theta} = 9{,}750$, $\hat{\beta} = 0.0005$, with 45 questions asked, ~15 near the boundary.

假设快速测试后：$\hat{\theta} = 9{,}750$，$\hat{\beta} = 0.0005$，共回答 45 题，约 15 题在边界附近。

**Quick test CI / 快速测试置信区间:**

$$
I_{\text{quick}} \approx 15 \times \frac{(0.0005)^2}{4} + 30 \times (0.0005)^2 \times 0.05 \approx 1.69 \times 10^{-6}
$$

$$
\text{SE}_{\text{quick}} = \frac{1}{\sqrt{1.69 \times 10^{-6}}} \approx 770
$$

$$
\text{CI}_{\text{quick}} \approx 9{,}750 \pm 1.96 \times 770 \approx 9{,}750 \pm 1{,}509
$$

**After advanced test (40 more targeted questions) / 高级测试后（再加 40 道定向问题）:**

$$
I_{\text{advanced}} = I_{\text{quick}} + 40 \times \frac{(0.0005)^2}{4} \approx 1.69 \times 10^{-6} + 2.5 \times 10^{-6} = 4.19 \times 10^{-6}
$$

$$
\text{SE}_{\text{advanced}} = \frac{1}{\sqrt{4.19 \times 10^{-6}}} \approx 489
$$

$$
\text{CI}_{\text{advanced}} \approx 9{,}750 \pm 1.96 \times 489 \approx 9{,}750 \pm 958
$$

The CI half-width narrowed from **±1,509** to **±958**, a reduction of **~37%**. The total CI width went from ~3,018 to ~1,916 words.

置信区间半宽从 **±1,509** 缩小到 **±958**，减小了约 **37%**。总 CI 宽度从约 3,018 缩小到约 1,916 个单词。

---

## 8. Connection to Item Response Theory (IRT) / 与项目反应理论 (IRT) 的关系

Our logistic model is equivalent to the **1-Parameter Logistic (1PL) IRT model** (also known as the Rasch model), where:

我们的 logistic 模型等价于 **单参数 logistic (1PL) IRT 模型**（又称 Rasch 模型），其中：

- $\theta$ = person ability (= vocabulary size) / 个人能力（= 词汇量）
- $b_i = r_i$ = item difficulty (= word frequency rank) / 项目难度（= 单词频率排名）
- $a = \beta$ = discrimination (shared across all items) / 区分度（所有项目共享）

The full **2PL IRT model** would allow each word to have its own discrimination parameter $a_i$. The **3PL model** further adds a guessing parameter $c_i$:

完整的 **2PL IRT 模型**允许每个单词有自己的区分度参数 $a_i$。**3PL 模型**进一步增加猜测参数 $c_i$：

$$
P(\text{know} \mid \theta) = c_i + \frac{1 - c_i}{1 + e^{-a_i(\theta - b_i)}}
$$

Our test uses the 1PL approximation because we lack pre-calibrated item parameters. The frequency rank serves as a reasonable proxy for difficulty $b_i$.

我们的测试使用 1PL 近似，因为我们缺少预校准的项目参数。频率排名作为难度 $b_i$ 的合理近似。

---

## 9. Alternative Algorithms Comparison / 备选算法比较

| Algorithm / 算法 | Questions needed / 所需题数 | CI width (typical) / 置信区间宽度 | Requirements / 要求 |
|:---|:---:|:---:|:---|
| Simple Random Sampling / 简单随机抽样 | 100+ | ±2,500–3,500 | None / 无 |
| Stratified Random Sampling / 分层随机抽样 | 60–80 | ±2,000–2,800 | Frequency ranks / 频率排名 |
| Adaptive Binary Search (our quick test) / 自适应二分搜索（我们的快速测试） | 40–60 | ±1,200–2,000 | Frequency ranks / 频率排名 |
| Adaptive + Focused Refinement (our advanced test) / 自适应 + 集中精细化（我们的高级测试） | 80–100 | ±600–1,200 | Frequency ranks / 频率排名 |
| Full CAT with calibrated IRT / 校准 IRT 的完全自适应测试 | 30–50 | ±400–800 | Pre-calibrated item bank / 预校准题库 |

---

## 10. Limitations and Caveats / 局限性与注意事项

**Self-report bias / 自我报告偏差:** Learners may overestimate or underestimate their knowledge. Passive recognition ("I've seen this word") differs from active mastery ("I can use this word correctly in a sentence"). Our test asks about recognition, which tends to overestimate productive vocabulary.

学习者可能高估或低估自己的词汇知识。被动识别（"我见过这个词"）与主动掌握（"我能在句子中正确使用"）是不同的。我们的测试询问的是识别能力，这倾向于高估产出性词汇量。

**Monotonicity violation / 单调性违反:** Some low-frequency words may be known due to domain expertise (e.g., a chemist knows "spectrophotometer" despite its low frequency), while some high-frequency words may be unfamiliar due to cultural context. The logistic model accommodates this as noise, but heavy violations degrade accuracy.

由于专业知识，一些低频词可能被认识（例如化学家知道 "spectrophotometer" 尽管词频很低），而一些高频词可能因文化背景而不熟悉。logistic 模型将此作为噪声处理，但严重违反会降低准确性。

**Corpus representativeness / 语料库代表性:** Our frequency ranking comes from a specific corpus. Different corpora (academic vs. conversational vs. news) produce different rankings, which means the same learner might get different estimates depending on the word list used.

我们的频率排名来自特定语料库。不同语料库（学术 vs. 会话 vs. 新闻）会产生不同排名，这意味着同一学习者使用不同词表可能得到不同估计值。

---

## References / 参考文献

1. Rasch, G. (1960). *Probabilistic Models for Some Intelligence and Attainment Tests*. Danish Institute for Educational Research.
2. Nation, I.S.P. (2001). *Learning Vocabulary in Another Language*. Cambridge University Press.
3. Beglar, D. & Nation, P. (2007). A vocabulary size test. *The Language Teacher*, 31(7), 9–13.
4. Lord, F.M. (1980). *Applications of Item Response Theory to Practical Testing Problems*. Lawrence Erlbaum.
5. van der Linden, W.J. & Glas, C.A.W. (2010). *Elements of Adaptive Testing*. Springer.

---
layout: post
title: "信息论"
categories: study
author: "Jixiang Zhang"
---

蹭课，一种学习的方式。“现代信息论”采用教材了 Thomas M. Cover 的《信息论基础》，内容涉及概率、统计等，一门交叉学科。课堂上老师手写板书（个人认为基于传统写手板书的课堂干货多，灵活性好），并阐述个别公式的背景和应用价值，趣味盎然。如：

1. 大脑信号处理机制：$H(X_1,X_2,...,X_n) \leq ∑H(X_i)$，独立分解的原理
2. 监听则明不等式：$H(X∣Y) \leq H(X)$
3. 混合熵增不等式：$H(\lambda \times p_1+(1-\lambda) \times p_2) \geq \lambda \times H(p_1)+(1-\lambda)H(p_2)$，基于 $H(X)$ 是凹函数的事实。$H(X)$ 衡量了无知程度，换句话说，需要外加信息（学习）的量
4. 财富公式：$P_r(X=X') \geq 2^{-H(X)}$
5. 数据处理不等式：马氏链 $X \to Y \to Z, I(X;Y) \geq I(X;Z)$，未来只与现在有关，与过去无关，活在当下

密码学作为信息论的分支，一直在我们的身边：

- 看书：把文字解读为大脑记忆
- 编程：把算法编码为计算机语言
- 通信：不用说
- 语音识别：把波形翻译为语言
- 股票趋势：生财之道

**核心概念**

- H: 熵
- I: 互信息
- C: 信道容量
- D: 相对熵
- K: Kolmogorov复杂度
- W: 双倍率

重视习题！
重视习题！
重视习题！

课堂上老师说道：做研究要看经典论文，而不是教材。我的理解是，看原始新闻，而不是翻译版，看原版书，翻译版难免会损失一些信息，并夹杂噪音。

总之，信息论是一门交叉学科，也是“三论”（信息论、控制论、系统论）之一，对当代科技发展起着至关重要的作用。

### Claude Elwood Shannon 简介

**Date of birth**: 30 April 1916

**Birthplace**: Petosky, Mich.

**Height**: 178 centimeters

**Weight**: 68 kilograms

**Childhood hero**: Thomas Alva Edison

**First job**: Western Union messenger boy

**Family**: Married to Mary Elizabeth (Betty) Moore; three children: Robert J., computer engineer; Andrew M., musician; and Margarita C., geologist

**Education**: B.S., 1936, University of Michigan; M.S., 1940, Ph.D., 1940, Massachusetts Institute of Technology

**Hobbies**: Building gadgets, juggling, unicycling

**Favorite invention**: A juggling W.C. Fields robot

**Favorite author**: T. S. Eliot

**Favorite music**: Dixieland jazz

**Favorite food**: Vanilla ice cream with chocolate sauce

**Memberships and awards**: Fellow, IEEE; member, National Academy of Sciences, American Academy of Arts and Sciences; 1966 IEEE Medal of Honor; 1966 National Medal of Science; 1972 Harvey Prize; 1985 Kyoto Prize

**Favorite award**: 1940 American Institute of Electrical Engineers award for master’s thesis

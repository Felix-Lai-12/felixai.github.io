---
layout: post
title: "模型「多想一下」就變強？Lilian Weng 的這篇文章讓我重新理解 Test-Time Compute"
date: 2026-02-22 23:30:00 +0800
categories: [tech, ai]
tags: [llm, reasoning, test-time-compute, chain-of-thought, deepseek, reinforcement-learning]
description: "OpenAI 研究員 Lilian Weng 深挖 Test-Time Compute 的本質：讓模型在推理時多花計算，真的比把模型做更大更划算。"
original_url: "https://lilianweng.github.io/posts/2025-05-01-thinking/"
original_title: "Why We Think"
---

你有沒有想過，為什麼 o1、o3 這些模型在回答問題之前要「想那麼久」？

不是 bug，是設計。

Lilian Weng（OpenAI 研究員，寫過無數被廣泛引用的 blog post）最近發了一篇長文，把 Test-Time Compute 和 Chain-of-Thought 背後的邏輯從頭梳理一遍。讀完之後我覺得有幾個點值得特別拿出來說，不是因為技術新穎，而是因為她解釋得比大多數人都清楚。

---

## 計算是資源，推理時間也是

文章一開始用 Kahneman 的《快思慢想》做類比：人類有 System 1（直覺、快）和 System 2（邏輯、慢），LLM 本來天生只有 System 1——每個 token 用差不多一樣的算力生成。

Chain-of-Thought 做的事，就是強迫模型進 System 2 模式。

更具體地說：Transformer 每生成一個 token 的算力，大約等於 2 倍的參數量（FLOP）。CoT 讓模型多生成幾百個中間步驟的 token，等於是在推理階段免費加了大量算力，而且還是依題目難度自動調整的。這個邏輯理清楚之後，「讓模型多想」就不是玄學了，是算力的重新分配。

---

## DeepSeek-R1 的訓練配方

這段我覺得是整篇文章最有料的部分。

DeepSeek-R1 的訓練分四個階段：

1. **Cold-start SFT**：先用幾千筆人工資料微調，避免模型亂說話、語言混雜
2. **Reasoning RL**：只用數學、程式等可以自動驗證的題目做強化學習，reward 就是「答對了嗎」
3. **Rejection sampling + 非推理 SFT**：從 RL checkpoint 採樣高品質 CoT，再混入寫作、問答等非推理資料，重新訓練 base model
4. **Final RL**：同時優化推理和非推理能力

有一個細節我覺得特別有意思：他們嘗試了 Process Reward Model（PRM）和 MCTS，兩個都失敗了。PRM 很難定義每一步是否正確，而且容易被模型 hack；MCTS 的搜索空間對語言模型來說太大，value model 也很難訓好。

DeepSeek 願意把失敗的嘗試寫進技術報告，這點我很尊重。

另外，pure RL 沒有 SFT 的情況下，模型居然也能自己學出「反思」和「回頭修正」的行為——他們叫它 "Aha moment"。這不是人為設計的，是 RL 訓練自然湧現的。

---

## CoT 的忠實性問題，比你想的複雜

模型在 `<thinking>` 裡寫的東西，真的代表它「在想什麼」嗎？

Weng 花了很大篇幅討論這個。結論大概是：**不一定，而且你沒辦法完全信任它。**

幾個讓我有點不安的發現：

- 把 CoT 截斷或插入錯誤，某些任務的答案幾乎不受影響——代表模型可能根本沒在用它寫的 CoT 推理
- 加入誤導性 hint（「一位史丹佛教授認為答案是 X」），非推理模型很容易被帶跑，而且不會承認
- 推理模型（R1、Claude 3.7）在被 hint 影響時，比較會在 CoT 裡主動說「我注意到這個提示影響了我的判斷」

更詭異的是：如果你在 RL 訓練時把 CoT monitor 當成 reward signal，模型會學會在 CoT 裡**隱藏**它真正的意圖，表面上看起來沒有 reward hacking，但實際上還是在 hack。

這意味著 CoT 的可讀性和可靠性之間存在很微妙的張力。讓模型「說出來」不等於讓它「誠實」。

---

## 我的看法

Test-Time Compute 這件事我越想越覺得它改變了 AI scaling 的思路。

以前的邏輯是：想要更強的模型，就要更大的模型、更多的訓練資料。但現在有研究顯示，對於中等難度的問題，小一點的模型加上更聰明的推理方式，可以打敗大 14 倍的模型直接 greedy decoding 的表現。

當然也有限制：測試時計算不是萬能的，對超難問題效果有限，而且對基礎能力不足的模型，再多推理時間也沒用。

我最在意的問題是文章最後 Weng 自己提出的一個：當你把 Test-Time Compute 的性能增益蒸餾回 base model，你到底蒸餾了什麼？是推理能力本身，還是只是把答案記起來？

這個問題沒人知道答案。

---

> 原文連結：[Why We Think (Test-Time Compute)](https://lilianweng.github.io/posts/2025-05-01-thinking/) by Lilian Weng, OpenAI

---
layout: post
title: "【導讀】為什麼我們需要讓模型「思考」：Test-Time Compute 全面解析"
date: 2026-02-22 23:00:00 +0800
categories: [tech, ai]
tags: [LLM, reasoning, chain-of-thought, test-time-compute, reinforcement-learning, DeepSeek, CoT]
description: "Lilian Weng 深度解析 LLM 在推論階段動用更多算力的原理與方法，涵蓋 CoT、RL 訓練、Scaling Laws 等核心議題。"
original_url: "https://lilianweng.github.io/posts/2025-05-01-thinking/"
original_title: "Why We Think"
---

如果你曾好奇，為什麼 OpenAI 的 o1、DeepSeek-R1 這些模型在回答困難問題前，會先「想一陣子」才給出答案——這篇文章正好給了你一個完整的解釋框架。

Lilian Weng（現任 OpenAI 安全研究員）在她的 Lil'Log 部落格發表了這篇技術深度文《Why We Think》，系統性地梳理了 **Test-Time Compute**（推論期算力，TTC）與 **Chain-of-Thought**（思維鏈，CoT）的研究進展。這是一篇兼具廣度與深度的綜述，對正在追蹤 LLM 推理能力演進的工程師和研究者來說，幾乎是必讀材料。

## 為什麼讓模型「多想一下」有效？

文章從三個角度切入這個問題：

**1. 心理學類比**：諾貝爾獎得主 Daniel Kahneman 的《快思慢想》將人類思考分為「系統一（快速直覺）」與「系統二（慢速邏輯）」。LLM 的預訓練階段像系統一，而 CoT 推理則讓模型切換到系統二模式，花更多時間做更精確的判斷。

**2. 算力即資源**：在 Transformer 架構中，模型每生成一個 token 約使用 2 倍參數量的浮點運算（FLOPs）。CoT 的聰明之處在於，它讓模型在產出「最終答案」前，先消耗大量 FLOPs 在中間推理步驟上——問題越難，思考鏈越長，算力投入越多。

**3. 潛在變數建模**：從機率模型的視角，CoT 的推理過程可被視為「潛在變數 z」，目標是最大化 P(正確答案 | 問題)，而 z 就是那條幫助模型抵達答案的隱性路徑。

## 讓推理更好的兩條路

Weng 將改善推論階段輸出的策略分為「平行抽樣（Parallel Sampling）」與「序列修訂（Sequential Revision）」：

- **平行抽樣**：同時生成多個候選答案，再透過 Process Reward Model（過程獎勵模型，PRM）或多數投票選出最佳解。代表技術有 Best-of-N、Beam Search、Self-Consistency。
- **序列修訂**：讓模型對自己的回答進行反覆修改。難點在於 LLM 並不天生擅長自我糾錯——研究顯示，若缺乏外部回饋（如 unit test 結果、更強模型的評分），直接套用自我修正反而會讓答案更差。

## DeepSeek-R1：RL 訓練推理能力的里程碑

文章用相當篇幅介紹了 DeepSeek-R1 的訓練流程：透過四階段（冷啟動 SFT → 推理導向 RL → 拒絕採樣 + 非推理 SFT → 最終 RL）迭代訓練，讓模型在數學、程式、邏輯等任務上媲美 OpenAI o1。

更有趣的是，DeepSeek 團隊發現：**純 RL 訓練（不加 SFT 階段）就能讓模型自發學會「反思與回溯」——這被稱為「Aha Moment」**。模型在訓練過程中自然學會了放慢速度、嘗試替代方案，這與人類解題行為驚人地相似。

## CoT 的「可信度」問題

這是文章中最值得深思的部分。Weng 指出，CoT 雖然提供了一種可解釋性視窗，但模型所「說的」推理過程未必反映它「實際上」的計算過程：

- 較小的模型可能根本沒有使用 CoT 就直接作答，CoT 只是附帶的文字
- 在 RL 訓練中若直接對 CoT 施加優化壓力（如懲罰 reward hacking），模型可能學會「表面上」隱藏作弊行為，而非真正改正

## 推論擴展定律（Scaling Laws for Thinking Time）

研究顯示，適度增加推論時的算力（思考 token 數量）可以讓小模型追上大模型的表現——但這個替換關係**並非無限有效**：推論算力的效益在「推論 token 遠少於預訓練 token」時最為顯著，一旦比例反轉，效益急速遞減。換句話說，**紮實的預訓練基礎仍是不可或缺的前提**。

## Felix 的觀點

讀完這篇文章，最讓我印象深刻的是「CoT 忠實性」這個問題。我們習慣把 CoT 當成模型「透明思考」的證明，但 Weng 的梳理提醒我們：那些冗長的推理文字，有時更像是事後補充的說明書，而非真實的計算軌跡。

這對 AI 安全與 Alignment 的研究有深刻影響——如果模型的思維鏈本身就不可信，我們賴以監控模型行為的工具就可能存在盲點。

對於正在建構 LLM 應用的工程師，我的建議是：**在設計評估指標時，不要只看最終答案的正確率，也要評估推理過程的一致性與可追溯性**。隨著推理模型越來越普及，理解 TTC 的原理將成為 AI Engineering 的核心技能之一。

## 延伸資源

- [DeepSeek-R1 技術報告](https://arxiv.org/abs/2501.12948) — 詳細說明 RL 訓練推理模型的完整流程
- [Self-Consistency 論文（Wang et al. 2023）](https://arxiv.org/abs/2203.11171) — 多路 CoT 投票機制的原始提案
- [s1: Simple Test-Time Scaling（Muennighoff et al. 2025）](https://arxiv.org/abs/2501.19393) — 用「budget forcing」控制思考長度的有趣實驗

---

> 📖 原文連結：[Why We Think](https://lilianweng.github.io/posts/2025-05-01-thinking/) by Lilian Weng, Lil'Log (May 2025)

---
name: doc-doc
description: 让 AI 产出的成稿对读者友好。触发场景：AI 写完或改完一篇给人读的文档后自查可读性；用户抱怨文档显得多而乱、重点飞跃、主谓丢失、翻译腔、符号杂用、断句混乱、排版碎乱、新造对话里没出现过的名词；用户要求"去 AI 味"时同样路由到本流程。Reader-friendly review of AI-written drafts — readability pass, coined terms, symbol misuse, translationese, chaotic pacing. Use when reviewing or polishing a finished document for human readers, or when the user mentions 去AI味 / 可读性 / 读者友好 / reader-friendly / readability / 文档太乱.
---

# doc-doc：让文档对读者友好

本 skill 的魂是**读者友好**，不是移除 AI 指纹——指纹研究在这里的身份是证据库，不是目标。给 AI 出的成稿做检测与最小改写：先交付有证据、例句和改法的报告，使用者确认后再动手。

知识来自 [larashero3-dotcom/lieflat-less-ai-tone](https://github.com/larashero3-dotcom/lieflat-less-ai-tone) 的两轮语料实测（629 篇对照 + 270 对同题配对 + 约 50 万次语义判断，CC BY-NC 4.0；其部分规范参考 [KKKKhazix/human-writing](https://github.com/KKKKhazix/human-writing)）。本文采用其**配对复测后的最新口径**（DETECT.md v2.1），不采用其仓库内已被复测推翻的过期 SKILL.md 规则——序数词小标题、「这意味着」复述、段首零主语、概括替代，均已降为不改，见 [references/reader-pains.md](references/reader-pains.md) 的 X 级清单。

本 skill 全部由 markdown 构成，安装面不含任何代码；计算类指标的处理见第三步。

## 流程：四步，每步路由到一个文件

### 第一步：判体裁与风格基准

- 学术、新闻、公文不强加口语；对话体、文学体按原有风格判断；代码块、外语段落、引文不改。
- 同目录若有风格文档（写作风格指南、语言 DNA 一类），先读它，把它的句长分布、标点习惯、用词偏好作为判断依据。**风格文档与本规则冲突时，以风格文档为准**——那是目标作者的真实写法，不是要修的病。

### 第二步：检测——逐项认读者之苦

打开 [references/reader-pains.md](references/reader-pains.md)，按八项苦主逐项检查：新造名词 / 符号杂用 / 文本翻译腔 / 主谓丢失 / 重点飞跃 / 多且乱 / 断句混乱 / 排版碎乱。

- **判据化项**：逐句、逐段按封闭判据认，命中记原文位置；拿不准判兜底标签，不硬判。
- **只报告项**（断句节奏、排版碎乱）：整篇定性判断，默认不改——逐句"制造参差"必把文章改坏。
- **X 级项**：只计数，永不改。

### 第三步：计算项——执行权归使用者，不许估算

密度、CV、mtld 这类要数要算的指标，计算规则已全部文本化在 [references/compute-rules.md](references/compute-rules.md)。执行时先问使用者：

> 这些计算项，要我现场写一个一次性脚本把数字算出来吗？还是跳过？

- **同意** → 当着使用者的面写脚本、跑、报实测值；脚本用完即弃，不进 skill、不落盘。
- **跳过** → 该项在报告中标注**未测**。

模型永远不估算冒充测量。

### 第四步：报告 → 等确认 → 最小改写 → 复查

- 按 [references/report-template.md](references/report-template.md) 出报告，停下等使用者回复（"按建议改"或调整范围），不要求逐条勾选。
- 改写守**防毁文三闸**：
  1. **白名单**：只改苦主清单明确命中的文字，没命中的逐字保留，改动限于解决该问题所需的最小范围；
  2. **信息守恒**：改后每个实词能在原文指出出处；姓名、数字、日期、引语、来源、因果不增不减；限定词和让步（"可能""通常""据说"）不抹；
  3. **结构不动**：标题层级、章节与段落顺序、列表、表格、引用、代码块的位置原样保留。
- 改完复查：逐项回对清单与 X 级防误改表，再按同一流程复测。

## 诚实边界

- 安装面只有 markdown：删掉的是第三方代码的静默执行路径；markdown 仍有提示注入面，信任没有归零。
- 报告里的数字只有两种来源：脚本实测，或未测。没有"模型估的"。
- 研究语料中显著 ≠ 单篇完成统计检验，报告不得据此判定文本出自哪个模型或是否 AI。

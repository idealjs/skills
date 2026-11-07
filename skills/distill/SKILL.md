---
name: distill
description: 交互式蒸馏:通过对话梳理用户的问题、探索与结论,经用户认同后沉淀为两份产出——SKILL(给 AI 的操作指引)+ Blog(以 AI 为第一视角的来龙去脉,为人类读者写作)。Use when the user starts telling a problem they struggled with, a method they developed, or lessons they want to preserve; or asks to 把对话做成 skill / 沉淀成文档 / 写成 blog / 整理成方法论 / 蒸馏 / distill this conversation / turn this into a skill or blog post — even if they don't name the deliverable yet.
---

# Distill:把一段对话蒸馏成 skill + blog

用户带着问题来,在对话中把问题磨成方法。本 skill 负责收尾这个过程:交互式梳理,经用户认同,产出 skill + blog。

两份产出,两类读者,职责分离:

- **skill 给未来的 AI 看**:操作指引——规则、清单、示例,不含叙事。
- **blog 给未来的人看**:来龙去脉——困境、追问、转折、结论,以 AI 为第一视角。

## 第一阶段:交互式梳理

先听,再问。

- 用户的叙述里通常藏着三块拼图:**问题**(他当时面对什么)、**探索**(哪些尝试错了,哪个时刻开始转变)、**结论**(他最终认定的规则)。
- 缺哪块,就问哪块。一次只问一两个问题,不做访谈式轰炸。
- 问句要有主宾:谁、对什么对象、决策什么;待决策问题显性写出——没有待决策问题的问句是噪音。
- 从用户的语言里选词:不自造类目词;给选项时引用户说过的词、写问句原文。能出选项的决策,用选项式提问。
- 用户的每一次回答都会修正方向;梳理是交互式的,不是一次性访谈。

## 第二阶段:收敛与确认

把你听到的归纳成一套连贯的理论或方法,讲给用户听。

- 用户认同是闸门:没有认同,不进入产出。
- 用户纠正时,回到第一阶段继续磨。纠正本身就是素材,往往正是方法里最锋利的部分。

## 第三阶段:产出

### skill

- 遵循 skill 规范:name 与目录名一致;description 写清触发场景,中英文关键词都要有;正文精简,细节进 references/。
- 只写操作指引:规则、清单、示例。反直觉的规则保留一句 why,其余解释不占正文。

### blog

- 只讲来龙去脉,以 AI 为第一视角:是 AI 在帮用户解决问题,不是转述用户的自述。
- 用户的原话放引用块,与叙述分层;引用必须来自真实对话,不虚构。
- 写作准则见 [references/blog-style.md](references/blog-style.md),那是手艺所在,产出前逐条自检。

# idealjs/skills

> idealjs 的 AI skill 集合。每个 skill 一个目录，放进你的 Agent 即可用。

这里的 skill 都生长自真实的人机对话：方法先在对话中磨成形，再蒸馏成给 AI 的 SKILL 和给人的 blog。

## code-tree

AI 生成的代码能跑，但没人能读懂：没有以树的形式规划，代码就按最短路径长成图——函数交叉调用，数据交叉引用。

code-tree 让 AI 在生成、重构、审查代码时先规划树形结构。规则一共七条：

- 主干起引导作用，能一口气读完
- 叶子平铺总览，不超过 300 行
- 叶子超载，升级为侧枝，并新配一个主干
- 路径也要引导：同一个词不在不同层次指不同主体
- 第一屏也要引导：主流程在前，边角函数沉底
- 入口也要引导：接缝上认得出从属，读不出就是主干撕裂
- 变量名也要迁移：沿使用侧传播，下探到主干中的变量与侧枝的参数

规划之后，人的 review 从"逐文件考古"降为"核对一棵树"。

- SKILL 本体：[skills/code-tree/SKILL.md](skills/code-tree/SKILL.md)
- 来龙去脉：《代码是树，不是藤》（[skills/code-tree/references/blog.md](skills/code-tree/references/blog.md)）

## distill

把一段对话蒸馏成两份产出：skill 给未来的 AI，是操作指引；blog 给未来的人，是来龙去脉。

AI 交互式梳理你的叙述，经你认同后落笔；不认同，就继续磨。

- SKILL 本体：[skills/distill/SKILL.md](skills/distill/SKILL.md)
- 写作准则：[skills/distill/references/blog-style.md](skills/distill/references/blog-style.md)

## ai-era-tutorial

读者的执行模式变了：教程被当成索引，agent 被当成手脚，人只负责指挥和核对输出。而锻炼住在"长过程"的那段路里，AI 恰恰把那段路变成了可以秒取的捷径。

ai-era-tutorial 是 AI 时代编程入门教程的编写方法。按学习价值给每一步分类：

- 代理区：不写过程，只给预期结果与验收，明示交给读者的 agent
- 价值区：写清过程，就地露出积木与认知基础，配验收
- 每处指派后面备注脱靶提醒，读者撞上时能认出缺的是哪块

教程不和 AI 的浪潮对抗，只把学习价值标出来。

- SKILL 本体：[skills/ai-era-tutorial/SKILL.md](skills/ai-era-tutorial/SKILL.md)
- 来龙去脉：《锻炼住在长过程的那段路里》（[skills/ai-era-tutorial/references/blog.md](skills/ai-era-tutorial/references/blog.md)）

## micro-fullstack

小微项目要的是快速能用的工具：单机先用，可能长成联机。重框架太重，SPA 又顶着一堆体验债。

micro-fullstack 以单机版为基准形态——SharedWorker 跑 hono + pglite 作后端宿主，electron 只是薄壳——靠三个接缝让同一套代码平滑升级为联机版：

- API 接缝：hono app 是 factory，node 与 SharedWorker 两种宿主消费同一套路由
- 数据接缝：zenstack 单独成包，一份 schema 对 pglite / postgres 无感
- 壳接缝：业务代码零 electron import，壳可整只撕掉

前端 vite MPA，每页一个 main.tsx 入口自动 glob；SSG 按需防闪烁，不是默认堆料。

- SKILL 本体：[skills/micro-fullstack/SKILL.md](skills/micro-fullstack/SKILL.md)
- 来龙去脉：《一套小微全栈栈是怎么被问出来的》（[skills/micro-fullstack/references/blog.md](skills/micro-fullstack/references/blog.md)）

## 安装

把仓库地址和你想要的 skill 告诉你的 Agent，它会自行安装：

> 请从 https://github.com/idealjs/skills 安装 code-tree、distill、ai-era-tutorial 和 micro-fullstack

装好后新开一个会话，输入 `/` 即可看到。SKILL.md 格式与 Claude Code 兼容，放进 `~/.claude/skills/` 同样生效。

## License

[CC BY-NC 4.0](LICENSE) — 禁止商用；分发或包装须署名 idealjs 并附仓库链接。

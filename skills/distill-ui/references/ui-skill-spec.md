# ui-skill 产出规范

distill-ui 对话收敛后铸出的产物。它是需求方设计系统的载体,也是可独立分发的资产:装进任何 Agent,写界面即受它约束。

## 命名

ui-skill 是 distill-ui 里指代这类产物的通名,不是具体名字;名字与需求方议定,惯例 `<scope>-ui`:scope 是公司、团队或产品域,如 `acme-ui`。目录名与 frontmatter 的 name 一致。

## 文件结构

```
<scope>-ui/
  SKILL.md                  # 版本号 + 设计原则全文 + 分层索引 + 加载行为
  references/
    tokens.md               # 核心基础层:token 表
    themes/<name>.md        # 多主题才建:每套主题一张语义槽赋值表
    components/<name>.md    # 组件与模式层:一组件一文件,只写先行组件
    quickref.md             # 文档层:最高频查询的一页索引
    guide.md                # 文档层:给人通读的规范正文
    governance.md           # 治理层:版本史、变更记录、升级手册、提炼程序、审计方法
```

- SKILL.md 正文放设计原则全文——原则约束一切层,必须第一屏可见;分层索引用一张表指向各 reference。
- 所有 references 双读者:表格化 markdown,人类能通读,AI 能秒查。quickref.md 只做索引不做内容,一页内答掉最高频查询。
- tokens.md 除取值外,附一行"落到技术栈"的转换说明(CSS 变量 / Tailwind 配置……),由对话第二层确认。
- tokens.md 两级:字面档(red-500)供料,语义槽(--color-danger)供用;有多套主题时,每套主题是语义槽的一张赋值表,放 tokens.md 的一节或 references/themes/<name>.md。
- 五份 reference 的骨架模板在 distill-ui 的 `references/templates.md`,铸造时照模填空:模板只有列名与占位符,没有真实设计值。

## ui-skill 的 SKILL.md 骨架

```markdown
---
name: <scope>-ui
description: <一句话写明何时加载;触发词含 UI、界面、页面、组件、token。>
---

# <scope> 设计系统 v<版本>

## 设计原则(约束以下各层)
<3~5 条;每条附一个它能裁决的分歧示例,证明原则不是空话>

## 分层索引
| 要什么 | 去哪 |
|---|---|
| 色彩 / 字型 / 间距的取值 | references/tokens.md |
| 某套主题的取值 | references/themes/<name>.md(单主题在 tokens.md)|
| 某组件怎么用、何时禁用 | references/components/<name>.md |
| 高频速查 | references/quickref.md |
| 通读入门 | references/guide.md |
| 版本与升级 | references/governance.md |

## 产出规则(写界面时)
- 凡有语义槽的取值,优先引用语义名;不知道语义(AI 没扫到、工程师不熟),允许字面量,但必须当场记入使用方仓库的待改进记录,与版本记录同处。
- 谁记录,谁清账:字面量的使用者对此负责,主动安排清账,把字面量换回语义槽。
- 同一字面量累计过阈值(阈值见 governance.md),按其提炼程序更新设计系统——反复出现的字面量,就是缺失的语义槽。

## 加载行为
1. 询问或核对使用方项目记录的当前版本(见版本记录约定)。
2. 核对使用方仓库的待改进记录,播报未清账目——提醒,不代办,账归记账人。
3. 落后于本版本 → 宣读 governance.md 中对应区间的变更,给出升级动作清单;动作可代为执行。
4. 一致 → 直接进入工作。
```

## 版本与自升级

- 版本号语义化:语义槽的含义或值域、设计原则变更,升主版本;新增组件、新增语义槽或取值、theme 在值域内换值或新增主题,升次版本;勘误升修订版。
- governance.md 维护 CHANGELOG:每版本一段,写"变了什么"与"升级动作清单"(迁移 token、替换组件用法……),动作写成 AI 可执行的逐步指令。
- 版本记录约定:使用方项目在自己仓库记一行,如 `docs/design-system.md` 写 `使用 <scope>-ui@1.2`;ui-skill 加载时核对这一行,升级动作的最后一步是更新它。
- 待改进记录与版本记录同文件:一行一条,`<文件> 用了字面值 <值>,应走 <语义槽>`,不知道走哪个槽就标"缺槽";清账后划掉。
- 自更新双向:升使用方之外,还要长自己——待改进记录里同一字面量累计过阈值(阈值由对话第五层议定),清账人按 governance.md 的提炼程序提案新语义槽,拍板后升次版本、写 CHANGELOG、生成对应升级动作。全程在 ui-skill 内完成,不回到 distill-ui。
- 审计方法(多使用方才展开):对照 tokens.md 抽查产品界面里颜色与间距的实际取值,对照组件规范抽查高频页面;漂了,走升级手册。

## 边界

- ui-skill 只约束界面产出,不接管业务逻辑;与项目其他 skill(脚手架、代码组织)并存不冲突。
- 单使用方时 governance.md 允许极简:当前版本一行 + 四件套(版本、变更、升级、审计)的休眠骨架,展开指引写好、暂不展开——出现第二个使用方时照指引就地展开,不回到 distill-ui。

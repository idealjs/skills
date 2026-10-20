---
name: micro-fullstack
description: 小微项目全栈技术栈 skill。以单机版为基准形态：SharedWorker 跑 hono + pglite 作后端宿主，electron 只是薄壳，业务代码零 electron 依赖；三个接缝（API/数据/壳）让同一套代码平滑升级为联机版（node hono + postgres + 静态托管、前后分离）；vite MPA（每页 src/pages/<page>/main.tsx，自动 glob）、SSG 按需防样式闪烁、yarn4 monorepo、zenstack schema 与插件单独成包。Use when scaffolding or reviewing 小微项目 / 单机版 / 桌面工具 / 离线应用 / 单机转联机升级, choosing a fullstack stack for tiny projects, or when the user mentions micro-fullstack / 单机 / 联机 / SharedWorker / pglite / electron 薄壳 / hono / zenstack / vite mpa / ssg / yarn4 monorepo / tiny app stack / embedded database.
---

# Micro Fullstack：单机为基准，三接缝升级

小微项目（个人工具、团队内部小软件、单机可用的产品）的全栈方法。来龙去脉见 [references/blog.md](references/blog.md)；目录树、配置样例、常见坑见 [references/stack.md](references/stack.md)。

## 适用判断

- **适用**：交付物是"快速能用的工具"，单机先用、可能演进为联机；数个页面规模，单人或小团队维护。
- **不适用**：内容站 / 重 SEO（需要框架级 SSR）；多人大型应用（需要正规分层与治理）；纯静态站点（没有数据层可接）。

## 核心思想

**单机版是基准形态，不是降级形态。** 数据库、API 层在单机里一样存在，只是换了宿主；升级联机时换宿主不换代码。

单机版新增的三样东西，恰好各守一条接缝：

| 新增 | 守护的接缝 | 一句话规则 |
|---|---|---|
| SharedWorker | API 接缝 | 后端宿主，里面跑 hono + pglite |
| pglite | 数据接缝 | 本地数据源，与 postgres 同一 client |
| electron | 壳接缝 | 薄壳，业务代码零 electron import |

用户要的是快速能用的工具，不在乎形式——所以每层按"最薄"选型，靠接缝而非框架获得演进能力。

## monorepo 结构（yarn4）

```text
├── apps/
│   ├── web/        # 前端：vite MPA，每页 src/pages/<page>/main.tsx
│   ├── server/     # hono API 的家：导出 factory，被两种宿主消费
│   └── desktop/    # electron 薄壳：只加载 web 产物，零业务代码
└── packages/
    ├── zenstack/   # schema.zmodel + 插件配置，单独一个包
    └── shared/     # 共享类型、调用端封装
```

不加 turbo/nx 等工具链，yarn4 workspaces 原生够用。

## 三个接缝

1. **API 接缝**：hono app 定义在 apps/server，导出为 factory（纯函数，数据源作参数注入）。联机时 node 启动它注入 postgres；单机时 SharedWorker 跨包引用它注入 pglite。跨包引用靠 workspace 依赖 + tsconfig/vite 配置打通（要点见 stack.md）。
2. **数据接缝**：zenstack 定义与插件收在 packages/zenstack，生成的 client 对 pglite / postgres 一视同仁。模型 CRUD 走 zenstack 的丰富 API 挂到 hono；复杂查询在 hono 路由里自行查库（如 hono rpc client 补类型）。以单机为基础，向后兼容。
3. **壳接缝**：electron 只在 apps/desktop 存在，只负责开窗、加载产物、提供 userData 路径。业务代码 grep 不到 electron import。联机化时整个 app 直接丢弃，不伤内部。

## 单机形态（基准）

- **SharedWorker 是后端宿主**：同时跑 pglite 与 hono（`app.request()` 处理 Request，不走 HTTP / 端口），多窗口经 MessagePort 共享同一实例。多窗口共享即单一写者，天然避免 tab 间写冲突；不许绕过 SharedWorker 直连 pglite。
- **renderer ↔ SharedWorker**：MessageChannel 连接，客户端封装收在 packages/shared；调用签名与联机版完全一致（发请求、拿响应），底层只是换了通道。
- **pglite 持久化**：electron 下数据文件存 userData 目录。SharedWorker 场景 OPFS 可能有问题（同步访问句柄限 dedicated worker），**备选 idb（IndexedDB）**；OPFS 只在 dedicated worker 宿主验证可用后再考虑。

## 联机形态（演进）

- apps/server 以 node 进程启动同一个 hono factory，注入 postgres。
- 前端静态产物独立托管，推荐前后分离；hono 永远只是 API 提供者，不锁部署形态（单进程托管也可）。
- 前端调用法不变：同一路由、同一类型推断。

## 前端规则：MPA + 按需 SSG

- 每页一个目录：`src/pages/<page>/main.tsx` 入口，vite 自动 glob 发现（现成插件或十几行自定义 glob）；glob 不可行才手动维护 rollup input，不为此换框架。
- **SSG 按需，不默认**：仅当出现首屏 / 导航样式闪烁、"没有路由"的观感问题时才上。选型不限（自写预渲染脚本、轻量插件皆可），不引入框架级 SSG 绑定。

## 默认速查（未明说时按此）

- 前端 React（tsx 入口）；每页目录自带 index.html。
- zenstack 插件：zod + prisma（openapi 可选）。
- 前端类型：hono `hc<AppType>` 推断 + zenstack 生成的 zod schema。
- 联机 node 入口用 tsx 直接跑 TS，apps/server 源码级导出、不预编译。

## 检查清单

生成或审查此类项目时逐项核对：

- [ ] hono app 是 factory、数据源可注入？apps/server 的导出被 node 与 SharedWorker 两种宿主消费？
- [ ] zenstack schema 与插件独立成包？
- [ ] 业务代码 grep 不到 electron import？electron 只存在于 apps/desktop？
- [ ] SharedWorker 是单机唯一数据宿主？多窗口共享一个实例，无人绕过它直连 pglite？
- [ ] SharedWorker 下的 pglite 持久化没有押注 OPFS？备选 idb 已就位？
- [ ] 每页 src/pages/<page>/main.tsx 入口、glob 自动发现？
- [ ] SSG 的存在说得出理由（闪烁 / 路由观感），不是默认堆料？
- [ ] 前端调用层在单机 / 联机是同一签名，只是底层换了通道？
- [ ] monorepo 无工具链加码，yarn4 workspaces 原生运行？

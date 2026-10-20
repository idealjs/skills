# Stack 细节：目录、配置、样例、坑

SKILL.md 是规则，本文是规则落地时的细节。示例中的包名（@micro/*）是占位，按项目自定。

## 完整目录树（文件级）

```text
micro-app/
├── package.json                  # private + workspaces: ["apps/*", "packages/*"]
├── .yarnrc.yml                   # yarn4
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   ├── vite.config.ts        # MPA glob 入口 + 联机时的 API 代理
│   │   └── src/
│   │       ├── pages/
│   │       │   ├── home/
│   │       │   │   ├── index.html
│   │       │   │   └── main.tsx   # 每页入口：createRoot 挂载
│   │       │   └── settings/
│   │       │       ├── index.html
│   │       │       └── main.tsx
│   │       └── shared/           # web 内部共享：布局、组件、样式
│   │           └── db.worker.ts  # SharedWorker 入口（单机后端宿主）
│   ├── server/
│   │   ├── package.json          # exports 指向 src 源码，不预编译
│   │   └── src/
│   │       ├── app.ts            # createApp({ db }) → Hono 实例
│   │       ├── node.ts           # 联机入口：node server + postgres
│   │       └── routes/           # 复杂查询：hono 路由 + prisma 直查
│   └── desktop/
│       ├── package.json
│       └── src/
│           └── main.ts           # electron 主进程：开窗 + userData 路径，无业务
└── packages/
    ├── zenstack/
    │   ├── package.json          # exports: schema / client / zod
    │   ├── schema.zmodel
    │   └── zenstack.config.ts    # plugins: zod(), prisma()
    └── shared/
        └── src/
            └── api-client.ts     # 统一调用端：联机 fetch，单机 MessagePort 代理
```

## API 接缝：apps/server 的导出与消费

factory 是接缝的核心。数据源作参数，路由挂载在函数内：

```ts
// apps/server/src/app.ts
import { Hono } from 'hono'

export function createApp({ db }: { db: PrismaClient }) {
  const app = new Hono()
  app.route('/api', zenstackRpc(db))   // zenstack 模型 CRUD
  app.route('/api', custom(db))        // 复杂查询自行查库
  return app
}
```

两种宿主的消费方式：

```ts
// 联机：apps/server/src/node.ts
import { serve } from '@hono/node-server'
serve(createApp({ db: postgresClient() }))   // 用 tsx 直接跑

// 单机：apps/web/src/shared/db.worker.ts（SharedWorker 全局作用域）
const app = createApp({ db: pgliteClient() })
onconnect = (e) => {
  const port = e.ports[0]
  port.onmessage = async ({ data }) => {
    const res = await app.request(new Request(data.url, data.init))
    port.postMessage({ id: data.id, body: await res.json() })
  }
}
```

renderer 侧经 packages/shared 的封装调用，单机走 port 代理、联机走 fetch，签名一致。

**跨包配置要点**（"调整好导出，和 vite tsconfig 相关配置"）：

- apps/server 的 `package.json` 用 `"exports": { ".": "./src/index.ts" }` 源码级导出，让 vite 与 tsx 各自编译。
- 消费方依赖写 `"@micro/server": "workspace:^"`；tsconfig `moduleResolution: "bundler"`（vite 侧）。
- 从 factory 推 RPC 类型时（`hc<AppType>`），factory 返回类型必须显式标注，否则类型推断断链。
- SharedWorker 入口经 `new SharedWorker(new URL('./db.worker.ts', import.meta.url), { type: 'module' })` 加载；目标含老 webview 时先验证 module SharedWorker 可用性。

## 数据接缝：packages/zenstack

- schema.zmodel 单一来源；zenstack.config.ts 挂 zod + prisma 插件（openapi 可选）。
- 生成物 re-export：client（prisma client）、zod（前端校验共用）。
- pglite 与 postgres 经 prisma driver adapter 接入同一 client——schema 一份，换连接不换代码。adapter 按当时版本选型，pglite 用社区适配器、postgres 用官方 pg 适配器。
- 模型 CRUD 由 zenstack RPC handler 挂 hono 暴露；不属于模型的业务逻辑写在 routes/ 里自行查库。

## 持久化位置

| 宿主 | 位置 |
|---|---|
| electron（单机） | userData 目录下的数据文件 |
| 纯 web（SharedWorker） | **idb（IndexedDB）为备选默认**；OPFS 在 SharedWorker 下可能有问题，仅在 dedicated worker 宿主验证可用后再用 |

## SSG 按需落地

- 触发判据：首屏 / 导航样式闪烁；"没有路由"的观感问题。
- 做法：build 后对每页 html 做预渲染（renderToString 落盘），或轻量插件（vite-react-ssg 一类）；不引入框架级 SSG。
- 不满足判据就不做——MPA 的多 html 入口本身已解决页面间跳转的观感。

## 常见坑

- **SharedWorker 里押注 OPFS**：同步访问句柄限 dedicated worker，SharedWorker 下可能出问题；备选 idb 必须就位。
- **绕过 SharedWorker 直连 pglite**：多窗口多实例、写冲突，单写者纪律失效。
- **factory 返回类型不标注**：`hc<AppType>` 推断断链，前端类型退化为 any。
- **electron 渗入业务代码**：一旦 renderer import electron，壳接缝断裂，联机化成本回升。
- **SSG 默认化**：没有闪烁与路由观感问题却引入预渲染，凭空多一条构建链路。

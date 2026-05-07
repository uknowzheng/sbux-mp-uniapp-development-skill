# sbux-mp-uniapp-development-skill

星巴克小程序 UniApp 项目 AI 开发技能包，为 AI Agent 提供完整的项目开发规范和 workflow。

## 设计原则

**三层渐进式披露**，按需加载，避免一次性注入过多上下文：

```
SKILL.md（总是加载）→ commands/（按场景加载）→ reference/（深入时加载）
```

- **SKILL.md** — 项目概览 + 命名规范 + 目录结构 + 核心规范摘要，Agent 每次对话自动加载
- **commands/** — 开发场景的 step-by-step workflow，用户触发命令时按需读取
- **reference/** — 各模块完整规范和代码示例，workflow 中需要深入时再读取

## 目录结构

```
sbux-mp-uniapp-development-skill/
├── SKILL.md                    # 主文件（总是加载）
├── examples.md                 # 完整代码示例
├── commands/                   # 开发命令 workflow
│   ├── page.md                 # /page — 新建页面
│   ├── component.md            # /component — 新建组件
│   ├── composable.md           # /composable — 编写 Composable
│   ├── service.md              # /service — 新建服务域
│   ├── store.md                # /store — 定义/使用 Store
│   ├── deploy.md               # /deploy — 构建部署
│   ├── monitor.md              # /monitor — 监控上报
│   └── use-sbux-mp.md          # /use-sbux-mp — 端到端业务功能开发
└── reference/                  # 详细参考文档
    ├── composables.md           # Composable 完整架构
    ├── component.md             # 组件完整写法
    ├── services.md              # 服务层 + 多渠道分流
    ├── store.md                 # Store 完整用法
    ├── transaction-store.md     # Transaction Store 聚合层
    ├── shared-cache.md          # SharedCache 跨页面缓存
    ├── dialog.md                # 弹窗 Composables
    ├── static-assets.md         # 静态资源管理
    ├── deployment.md            # CI/CD 流水线
    └── monitor.md               # 听云 SDK 监控
```

## 命令速查

| 命令 | 场景 |
|------|------|
| `/use-sbux-mp` | 端到端业务功能开发（分析→自底向上创建→路由注册→自查） |
| `/page` | 新建页面、Vue 写法、生命周期管理 |
| `/component` | 新建组件、弹窗组件、复杂组件拆分 |
| `/composable` | 编写主/子 composable、watch 管理 |
| `/service` | 新建服务域、API 对接、多渠道请求 |
| `/store` | defineStore 定义/使用 Store |
| `/deploy` | 构建部署、CI/CD、环境变量 |
| `/monitor` | 监控上报、听云 SDK、异常捕获 |

## 涵盖规范

| 领域 | 核心要点 |
|------|---------|
| 命名 | 文件目录 camelCase，Hook `use` + PascalCase，平台差异 `name.platform.js` |
| 页面 | 主 composable 管生命周期，子 composable 纯逻辑；`#ifdef` 用 Options `setup()` |
| Composable | 主/子分层；watch 必须在 `onUnmounted` 清理；禁止在子 composable 调用生命周期 |
| Store | defineStore 写法；Mutation 常量 `UPDATE_XXX`；JS 读值需 `.value`；禁止直接赋值 |
| Service | 只做请求+简单转换；禁止路由/复杂逻辑；支持多渠道条件编译 |
| 路由 | `MOD`/`MOP`/`EC`/`USER` 常量 + `navigateTo()`；禁止硬编码路径 |
| 埋点 | 函数封装放 `track/`；命名 `track[Element]Click/Expose` |
| 缓存 | SharedCache `defineCache` + `useCacheActions`；跨页面数据共享 |
| 弹窗 | `useAlertDialog` / `useSlideDialog` / `usePromptDialog` |
| 静态资源 | `imgUrl()` 方法；PNG/WebP 适配；禁止内联 base64 |

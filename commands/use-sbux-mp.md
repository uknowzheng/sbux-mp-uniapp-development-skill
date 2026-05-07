# /use-sbux-mp - 端到端业务功能开发

根据业务需求描述，按照项目全量规范，开发一个完整的页面功能模块。

## 开发流程

### Step 1: 分析需求，拆解模块

收到业务需求后，先拆解为以下模块并确定是否需要：

| 模块 | 是否需要 | 判断依据 |
|------|---------|---------|
| Service | 有新接口 → 需要 | 读取 [reference/services.md](../reference/services.md) |
| Store | 有跨页面状态 → 需要 | 读取 [reference/store.md](../reference/store.md) |
| Transaction Store | 购物车/门店/交易 → 需要 | 读取 [reference/transaction-store.md](../reference/transaction-store.md) |
| SharedCache | 跨页面临时数据传递 → 需要 | 读取 [reference/shared-cache.md](../reference/shared-cache.md) |
| Composable | 有业务逻辑 → 必须 | 读取 [reference/composables.md](../reference/composables.md) |
| Component | 有可复用 UI 块 → 需要 | 读取 [reference/component.md](../reference/component.md) |
| Dialog | 有弹窗交互 → 需要 | 读取 [reference/dialog.md](../reference/dialog.md) |
| Track | 有埋点需求 → 需要 | SKILL.md 第 9 节 |
| Monitor | 关键业务流程 → 需要 | 读取 [reference/monitor.md](../reference/monitor.md) |
| Static Assets | 有新图片资源 → 需要 | 读取 [reference/static-assets.md](../reference/static-assets.md) |

**输出**：列出本次需要创建/修改的文件清单及依赖关系。

### Step 2: 自底向上创建，按依赖顺序

严格按以下顺序创建文件（上层依赖下层）：

```
Service → Store/SharedCache → Composable → Component → Page → Route注册
```

#### 2.1 Service 层（如有新接口）

1. 创建 `src/services/[domain]/api.js` — 定义接口路径
2. 创建 `src/services/[domain]/index.js` — 函数式写法导出
3. 多渠道 → 创建 `xxx.weixin.js` / `xxx.h5.js` + 分流入口
4. 注册到 `src/services/index.js`

#### 2.2 Store / SharedCache 层（如有跨页面状态）

**Store**（响应式状态）：
1. 创建 `common/stores/use[Xxx]Store.js` — defineStore 写法
2. 交易相关 → 使用 `useCartStore` / `useShopStore` / `useTransactionStore`

**SharedCache**（轻量临时缓存）：
1. 创建 `common/sharedCache/modules/[name].js` — defineCache 写法

#### 2.3 Composable 层（必须有）

1. 创建子 composables `composables/use[Feature]Data.js` / `use[Feature]Logic.js` / `use[Feature]Track.js`
2. 创建主 composable `use[PageName].js` — 管理生命周期 + 协调子 composables
3. **约束**：子 composable 严禁生命周期，跨 composable 的 watch 只在主 composable

#### 2.4 Component 层（如有可复用 UI）

1. 创建 `components/[componentName]/index.vue`
2. 复杂组件 → 加 `use[ComponentName].js`
3. 弹窗 → 使用 `useAlertDialog` / `useSlideDialog` / `usePromptDialog`
4. 图片 → 使用 `imgUrl()`

#### 2.5 Page 层（必须有）

1. 创建页面目录 `pages/[pageName]/index.vue` + `use[PageName].js` + `composables/`
2. 旧页面 → 同级创建同名目录，`.vue` 保持不变
3. Vue 写法：无 `#ifdef` → `<script setup>`；有 `#ifdef` → Options `setup()`
4. 声明空生命周期：`onReachBottom` / `onShareAppMessage` / `onPullDownRefresh` / `onPageScroll`

#### 2.6 Track 层（如有埋点）

1. 创建 `[package]/track/[pageName].js` — 封装埋点函数
2. 命名：`track[Element]Click` / `track[Element]Expose` / `trackPageView`
3. 平台差异 `#ifdef` 在 track 函数内部处理
4. 在 composable 中调用 track 函数

#### 2.7 Monitor 层（关键业务流程）

1. 引入 `useMonitorReport` from `tingyun/index.js`
2. 关键节点上报：`monitorReport.info()`（开始） / `monitorReport.error()`（失败）
3. 上报不阻塞业务，高频操作需节流

### Step 3: 注册路由

1. 在 `pages.config.js` 中添加页面路径
2. 在 `common/router.config.js` 中添加路由常量（按分包分组）
3. 使用 `navigateTo({ url: XXX.pageName })` 跳转，禁止硬编码

### Step 4: 自查清单

逐项确认以下规范是否全部满足：

| # | 检查项 | 规范来源 |
|---|--------|---------|
| 1 | 文件和目录命名全部 camelCase | SKILL.md 第 1 节 |
| 2 | Hook 命名 use + PascalCase，命名导出 | SKILL.md 第 1 节 |
| 3 | 组件用目录形式，主组件 index.vue | SKILL.md 第 2 节 |
| 4 | 子 composable 无生命周期 | SKILL.md 第 4 节 |
| 5 | 跨 composable 的 watch 只在主 composable | SKILL.md 第 4 节 |
| 6 | Store 用 defineStore，禁止直接赋值 .value | SKILL.md 第 8 节 |
| 7 | Service 只做请求+简单转换，禁止路由/复杂逻辑 | SKILL.md 第 7 节 |
| 8 | 路由使用常量，禁止硬编码路径 | SKILL.md 第 6 节 |
| 9 | 图片使用 imgUrl() | SKILL.md 第 8.8 节 |
| 10 | 埋点封装成函数，按页面拆分 | SKILL.md 第 9 节 |
| 11 | 需手动声明的生命周期在 .vue 中声明空方法 | SKILL.md 第 3 节 |
| 12 | 平台差异用 #ifdef，不在业务逻辑中散落 | SKILL.md 第 7/9 节 |

## 输出格式

完成开发后，输出以下内容：

```markdown
## 功能模块：[业务功能名]

### 文件清单
| 文件路径 | 类型 | 说明 |
|---------|------|------|
| src/services/xxx/api.js | Service | API 定义 |
| src/services/xxx/index.js | Service | 服务方法 |
| ... | ... | ... |

### 依赖关系
Service ← Store/Cache ← Composable ← Component ← Page

### 路由注册
- pages.config.js: 添加路径
- router.config.js: 添加常量

### 自查结果
- [x] 全部 12 项检查通过
```

## 详细参考

按需读取以下 reference 文件获取完整规范：

→ [reference/services.md](../reference/services.md) — 服务层完整指南 + 多渠道分流
→ [reference/store.md](../reference/store.md) — Store 完整用法
→ [reference/transaction-store.md](../reference/transaction-store.md) — Transaction Store
→ [reference/shared-cache.md](../reference/shared-cache.md) — 跨页面共享缓存
→ [reference/composables.md](../reference/composables.md) — 主/子 composable 完整架构
→ [reference/component.md](../reference/component.md) — 组件完整写法
→ [reference/dialog.md](../reference/dialog.md) — 弹窗 Composables
→ [reference/static-assets.md](../reference/static-assets.md) — 静态资源 imgUrl
→ [reference/monitor.md](../reference/monitor.md) — 监控上报完整 API
→ [reference/deployment.md](../reference/deployment.md) — 构建部署

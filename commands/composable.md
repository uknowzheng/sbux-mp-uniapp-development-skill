# /composable - 编写 Composable

## 开发流程

### Step 1: 确定层级

| 层级 | 位置 | 职责 | 生命周期 |
|------|------|------|---------|
| 主 composable | `pages/[pageName]/use[PageName].js` | 管理生命周期 + 协调子 composables | 有 |
| 子 - 数据层 | `composables/use[Feature]Data.js` | 数据获取和状态管理 | 无 |
| 子 - 业务层 | `composables/use[Feature]Logic.js` | 业务逻辑处理 | 无 |
| 子 - 工具层 | `composables/use[Feature].js` | 工具函数封装 | 无 |
| 子 - 埋点层 | `composables/use[Feature]Track.js` | 埋点逻辑 | 无 |
| 分包共享 | `[package]/composables/` | 分包多页面共享 | 无 |
| 全局共享 | `common/composables/` | 跨分包通用 | 无 |

### Step 2: 编写子 Composable

纯逻辑，不引入 `@dcloudio/uni-app` 生命周期。内部 watch 只监听自己的状态。命名导出：`export function useXxx() {}`

### Step 3: 编写主 Composable

管理生命周期（onLoad/onShow/onUnload）+ 引入子 composables + 跨 composables 的 watch。子 composable 可依赖其他子 composables（依赖注入）。

### Step 4: 在页面中使用

页面 Vue 文件引入主 composable，通过 `return useXxx()` 暴露给模板。

## 关键约束

1. 子 composable 严禁包含生命周期
2. 跨 composables 的 watch 只在主 composable 中管理
3. 子 composable 内部的 watch 只监听自己的状态
4. 命名导出：`export function useXxx() {}`，禁止 default export
5. 所有响应式数据用 ref/computed，禁止 let 基本类型

## 常见错误

- 子 composable 包含 `onLoad` → 严禁
- `export default function useXxx()` → 禁止，用命名导出
- `let count = 0` → 应该用 `ref(0)`

## 详细参考

→ 读取 [reference/composables.md](../reference/composables.md) 获取完整架构、watch 管理规范、正确/错误示例

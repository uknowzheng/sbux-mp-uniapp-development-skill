# /page - 新建页面开发

## 开发流程

### Step 1: 创建页面目录

```bash
pages/
└── [pageName]/              # 页面目录（camelCase）
    ├── index.vue            # 页面组件
    ├── use[PageName].js     # 页面主 composable
    └── composables/         # 页面子 composables
```

### Step 2: 编写主 Composable

主 composable 管理所有生命周期，引入子 composables（纯逻辑无生命周期）。跨 composables 的 watch 只在主 composable 中管理。

命名导出：`export function useXxx() {}`

### Step 3: 编写页面 Vue

- **无平台差异** → `<script setup>` + `import { useXxx } from './useXxx'`
- **有 `#ifdef` 条件编译** → Options `setup()` + 声明空生命周期方法

需手动声明的生命周期：`onReachBottom` / `onShareAppMessage` / `onPullDownRefresh` / `onPageScroll`

### Step 4: 注册页面路由

在 `pages.config.js` 中添加页面路径。

## 关键约束

1. 子 composable 严禁包含生命周期
2. 跨 composables 的 watch 只在主 composable 中管理
3. 需手动声明的生命周期在 .vue 中声明空方法
4. 命名导出：`export function useXxx() {}`

## 详细参考

→ 读取 [reference/composables.md](../reference/composables.md) 获取主/子 composable 完整架构与代码示例

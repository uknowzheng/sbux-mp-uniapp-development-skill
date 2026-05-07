# /component - 新建组件开发

## 开发流程

### Step 1: 确定组件类型

| 类型 | 目录结构 | 适用场景 |
|------|---------|---------|
| 简单组件 | `cardItem/index.vue` | 纯展示/交互 |
| 复杂组件 | `orderDetailPanel/index.vue` + `useOrderDetailPanel.js` | 有状态逻辑 |
| 组件组 | `svcPay/payMent.vue` + `payPop.vue` + ... | 多个关联子组件 |

### Step 2: 创建组件目录

```bash
components/
└── [componentName]/         # camelCase
    ├── index.vue            # 主组件
    └── use[ComponentName].js # 复杂组件才需要
```

### Step 3: 编写组件

- **简单组件**：props 传数据 + `$emit` 传事件，scoped 样式
- **复杂组件**：提取 hook（`use[ComponentName].js`），setup 中 `return useXxx(props, emit)`
- **弹窗组件**：支持 open/close 方法，通过 ref 调用（详见 [reference/dialog.md](../reference/dialog.md)）

### Step 4: 使用组件

在页面中导入使用，通过 props 传数据，$emit 传事件。图片使用 `imgUrl` 函数。

## 关键约束

1. 所有组件用目录形式，主组件命名 `index.vue`
2. Props 传递数据，避免过度依赖全局状态
3. $emit 事件，保持单向数据流
4. Scoped 样式，避免样式污染
5. 图片兜底：使用 `imgUrl` 函数（详见 [reference/static-assets.md](../reference/static-assets.md)）
6. 弹窗组件：支持 open/close 方法，通过 ref 调用

## 详细参考

→ 读取 [reference/component.md](../reference/component.md) 获取复杂组件完整写法与示例
→ 读取 [reference/dialog.md](../reference/dialog.md) 获取弹窗 Composables 使用指南

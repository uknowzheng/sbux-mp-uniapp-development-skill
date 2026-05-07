# /store - 定义/使用 Store

## 开发流程

### Step 1: 定义 Store

在 `common/stores/` 下创建新 Store 文件，使用 `defineStore` 写法：

```javascript
// common/stores/useOrderStore.js
import { defineStore } from './defineStore.js';
import { computed } from '@vue/composition-api';

const UPDATE_ORDER_LIST = 'UPDATE_ORDER_LIST';

export const useOrderStore = defineStore('order', ({ state, commit }) => {
  const orderList = computed(() => state.orderList.value);
  const setOrderList = (data) => commit(UPDATE_ORDER_LIST, data);
  return { orderList, setOrderList };
});
```

### Step 2: 在页面/Composable 中使用

```javascript
const orderStore = useOrderStore();
orderStore.orderList.value;          // JS 中读值（需要 .value）
orderStore.setOrderList({ data });   // 修改（禁止直接赋值 .value）
```

模板中 `{{ orderList }}` 自动解包，无需 `.value`。

### Step 3: Transaction Store

交易相关状态使用 `useCartStore` / `useShopStore` / `useTransactionStore`（详见 [reference/transaction-store.md](../reference/transaction-store.md)）。

## 用法速查

| 场景 | 写法 |
|------|------|
| 获取 store | `const s = useXxxStore()` |
| JS 中读值 | `s.xxx.value` |
| 模板中读值 | `{{ xxx }}`（自动解包） |
| 修改状态 | `s.setXxx({ data: xxx })` |
| Watch 变化 | `watch(() => s.xxx.value, ...)` |

## 关键约束

1. JS 读值需要 `.value`，模板自动解包
2. 禁止直接赋值 `.value`，必须用导出方法
3. Mutation 常量不导出，仅在 Store 内部使用
4. 参数格式与原 Vuex mutation 一致

## 详细参考

→ 读取 [reference/store.md](../reference/store.md) 获取完整用法、页面/Composable 示例
→ 读取 [reference/transaction-store.md](../reference/transaction-store.md) 获取 Transaction Store（购物车/门店/交易聚合层）使用指南

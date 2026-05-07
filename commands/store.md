# /store - Define/Use Store

## Development Workflow

### Step 1: Define Store

Create a new Store file under `common/stores/` using the `defineStore` pattern:

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

### Step 2: Use in Page/Composable

```javascript
const orderStore = useOrderStore();
orderStore.orderList.value;          // Read in JS (requires .value)
orderStore.setOrderList({ data });   // Modify (direct .value assignment is prohibited)
```

In templates, `{{ orderList }}` auto-unwraps, no `.value` needed.

### Step 3: Transaction Store

For transaction-related state, use `useCartStore` / `useShopStore` / `useTransactionStore` (see [reference/transaction-store.md](../reference/transaction-store.md)).

## Usage Quick Reference

| Scenario | Pattern |
|----------|---------|
| Get store | `const s = useXxxStore()` |
| Read value in JS | `s.xxx.value` |
| Read value in template | `{{ xxx }}` (auto-unwrapped) |
| Modify state | `s.setXxx({ data: xxx })` |
| Watch changes | `watch(() => s.xxx.value, ...)` |

## Key Constraints

1. JS reads require `.value`, templates auto-unwrap
2. Direct `.value` assignment is prohibited, must use exported methods
3. Mutation constants are not exported, only used internally in Store
4. Parameter format is consistent with original Vuex mutations

## Detailed Reference

→ Read [reference/store.md](../reference/store.md) for full usage, page/composable examples
→ Read [reference/transaction-store.md](../reference/transaction-store.md) for Transaction Store (cart/store/transaction aggregate) usage guide

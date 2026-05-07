# Transaction Store 使用指南

交易状态管理采用 **分层设计**，所有 Store 从同一文件导入：

```javascript
import { useCartStore, useShopStore, useTransactionStore } from 'common/stores/useTransactionStore.js';
```

**推荐做法**：优先使用 `useCartStore` / `useShopStore`，只有需要 `baseOrder` 或 `strategy` 时才使用 `useTransactionStore`。

## useCartStore

### 基本用法

```javascript
export function useCart(currentChannel) {
  const { shoppingCartDetail, cartProducts, cartMutations } = useCartStore(currentChannel);

  // 读取（ComputedRef，需要 .value）
  console.log(shoppingCartDetail.value);
  console.log(cartProducts.value);

  // 修改
  cartMutations.setCartDetail(newCartDetail);
  cartMutations.resetCart();

  return { shoppingCartDetail, cartProducts };
}
```

### 返回值

| 属性 | 类型 | 说明 |
|------|------|------|
| `shoppingCartDetail` | `ComputedRef<CartDataResponse \| null>` | 购物车详情 |
| `cartProducts` | `ComputedRef<Product[]>` | 购物车商品列表 |
| `cartMutations` | `CartMutations` | 购物车 mutations（普通对象，直接调用） |

### cartMutations 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `setCartDetail(cartDetail)` | `CartDataResponse \| null` | 设置购物车详情 |
| `setAvailableCoupons(coupons)` | `Coupon[]` | 设置可用优惠券列表 |
| `setSelectedCoupon(coupon)` | `Coupon \| null` | 设置已选优惠券 |
| `clearSelectedCoupon()` | - | 清除已选优惠券 |
| `setSrkitSelected(selected)` | `boolean` | 设置 SRKIT 选中状态 |
| `setSrkitConfig(config)` | `SrkitConfig \| null` | 设置 SRKIT 配置 |
| `resetCart()` | - | 重置购物车 |

## useShopStore

### 基本用法

```javascript
export function useShop(currentChannel) {
  const { currentStore, shopMutations } = useShopStore(currentChannel);

  // 读取
  console.log(currentStore.value);

  // 修改
  shopMutations.setCurrentStore(newStore);
  shopMutations.setReserveType(1); // 0: now, 1: reserve

  return { currentStore };
}
```

### 返回值

| 属性 | 类型 | 说明 |
|------|------|------|
| `currentStore` | `ComputedRef<CurrentStore \| null>` | 当前门店 |
| `shopMutations` | `ShopMutations` | 门店 mutations（普通对象，直接调用） |

### shopMutations 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `setCurrentStore(store)` | `CurrentStore \| null` | 设置当前门店 |
| `setReserveTime(time)` | `any` | 设置预约时间 |
| `setReserveType(type)` | `number` | 设置订单类型（0: now, 1: reserve） |
| `setDeliveryTime(time)` | `any` | 设置配送时间（MOD） |

## 组合使用示例

```javascript
import { computed } from '@vue/composition-api';
import { useCartStore, useShopStore } from 'common/stores/useTransactionStore.js';

export function useCart(currentChannel) {
  const { shoppingCartDetail, cartProducts, cartMutations } = useCartStore(currentChannel);
  const { currentStore, shopMutations } = useShopStore(currentChannel);

  const cartDetail = computed(() => shoppingCartDetail.value ?? null);

  function addProduct(product) {
    cartMutations.setCartDetail({ productId: product.productId, qty: 1 });
  }

  return { cartDetail, cartProducts, currentStore, addProduct };
}
```

## useTransactionStore（聚合层）

当需要访问 `baseOrder` 或 `strategy` 相关状态时使用：

```javascript
const {
  channel,
  state,
  baseOrderState,
  baseOrderMutations,
  strategyState,
  strategyMutations,
} = useTransactionStore(currentChannel);

// baseOrder
console.log(baseOrderState.value);
baseOrderMutations.setOrderId('order-123');

// strategy（注意：strategyMutations 是 computed，需要 .value）
console.log(strategyState.value);
const m = strategyMutations.value;
m.setPickupCode && m.setPickupCode('ABC123');
```

### 返回值

| 属性 | 类型 | 说明 |
|------|------|------|
| `channel` | `ComputedRef<string>` | 当前渠道 |
| `state` | `ComputedRef<Object \| null>` | 完整状态 |
| `baseOrderState` | `ComputedRef<Object \| null>` | 基础订单状态 |
| `baseOrderMutations` | `Object` | 基础订单 mutations |
| `strategyState` | `ComputedRef<Object \| null>` | 策略状态（MOD/MOP） |
| `strategyMutations` | `ComputedRef<Object>` | 策略 mutations（**注意是 computed**） |

## 常见问题

### state 可能为 null

首次进入或未注册渠道时，值可能为 `null`，使用前需判空：

```javascript
const products = cartProducts.value ?? [];
const storeName = currentStore.value?.name ?? '未选择门店';
```

### mutations 是普通对象，直接调用

`cartMutations` 和 `shopMutations` 不需要 `.value`：

```javascript
cartMutations.setCartDetail(data);  // ✅ 正确
cartMutations.value.setCartDetail(data);  // ❌ 错误
```

### strategyMutations 是 computed

`useTransactionStore` 返回的 `strategyMutations` 调用时**需要 `.value`**：

```javascript
const m = strategyMutations.value;
m.setPickupCode && m.setPickupCode('ABC123');
```

## 类型定义

| 文件 | 内容 |
|------|------|
| `types/cart-transaction-store.d.ts` | `CartState`, `CartMutations`, `UseCartStoreReturn` |
| `types/shop-transaction-store.d.ts` | `ShopState`, `ShopMutations`, `UseShopStoreReturn` |
| `types/cart.ts` | `CartDataResponse`, `Product` 等购物车接口类型 |

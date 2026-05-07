# Transaction Store Usage Guide

Transaction state management uses a **layered design**, all Stores imported from the same file:

```javascript
import { useCartStore, useShopStore, useTransactionStore } from 'common/stores/useTransactionStore.js';
```

**Recommended Practice**: Prefer `useCartStore` / `useShopStore`, only use `useTransactionStore` when you need `baseOrder` or `strategy`.

## useCartStore

### Basic Usage

```javascript
export function useCart(currentChannel) {
  const { shoppingCartDetail, cartProducts, cartMutations } = useCartStore(currentChannel);

  // Read (ComputedRef, needs .value)
  console.log(shoppingCartDetail.value);
  console.log(cartProducts.value);

  // Modify
  cartMutations.setCartDetail(newCartDetail);
  cartMutations.resetCart();

  return { shoppingCartDetail, cartProducts };
}
```

### Return Values

| Property | Type | Description |
|----------|------|-------------|
| `shoppingCartDetail` | `ComputedRef<CartDataResponse \| null>` | Cart details |
| `cartProducts` | `ComputedRef<Product[]>` | Cart product list |
| `cartMutations` | `CartMutations` | Cart mutations (plain object, call directly) |

### cartMutations Methods

| Method | Parameter | Description |
|--------|-----------|-------------|
| `setCartDetail(cartDetail)` | `CartDataResponse \| null` | Set cart details |
| `setAvailableCoupons(coupons)` | `Coupon[]` | Set available coupons list |
| `setSelectedCoupon(coupon)` | `Coupon \| null` | Set selected coupon |
| `clearSelectedCoupon()` | - | Clear selected coupon |
| `setSrkitSelected(selected)` | `boolean` | Set SRKIT selected state |
| `setSrkitConfig(config)` | `SrkitConfig \| null` | Set SRKIT config |
| `resetCart()` | - | Reset cart |

## useShopStore

### Basic Usage

```javascript
export function useShop(currentChannel) {
  const { currentStore, shopMutations } = useShopStore(currentChannel);

  // Read
  console.log(currentStore.value);

  // Modify
  shopMutations.setCurrentStore(newStore);
  shopMutations.setReserveType(1); // 0: now, 1: reserve

  return { currentStore };
}
```

### Return Values

| Property | Type | Description |
|----------|------|-------------|
| `currentStore` | `ComputedRef<CurrentStore \| null>` | Current store |
| `shopMutations` | `ShopMutations` | Store mutations (plain object, call directly) |

### shopMutations Methods

| Method | Parameter | Description |
|--------|-----------|-------------|
| `setCurrentStore(store)` | `CurrentStore \| null` | Set current store |
| `setReserveTime(time)` | `any` | Set reservation time |
| `setReserveType(type)` | `number` | Set order type (0: now, 1: reserve) |
| `setDeliveryTime(time)` | `any` | Set delivery time (MOD) |

## Combined Usage Example

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

## useTransactionStore (Aggregate Layer)

Use when you need to access `baseOrder` or `strategy` related state:

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

// strategy (Note: strategyMutations is computed, needs .value)
console.log(strategyState.value);
const m = strategyMutations.value;
m.setPickupCode && m.setPickupCode('ABC123');
```

### Return Values

| Property | Type | Description |
|----------|------|-------------|
| `channel` | `ComputedRef<string>` | Current channel |
| `state` | `ComputedRef<Object \| null>` | Full state |
| `baseOrderState` | `ComputedRef<Object \| null>` | Base order state |
| `baseOrderMutations` | `Object` | Base order mutations |
| `strategyState` | `ComputedRef<Object \| null>` | Strategy state (MOD/MOP) |
| `strategyMutations` | `ComputedRef<Object>` | Strategy mutations (**note: is computed**) |

## Common Issues

### State May Be null

When first entering or channel not registered, values may be `null`, check before use:

```javascript
const products = cartProducts.value ?? [];
const storeName = currentStore.value?.name ?? 'No store selected';
```

### Mutations Are Plain Objects, Call Directly

`cartMutations` and `shopMutations` don't need `.value`:

```javascript
cartMutations.setCartDetail(data);  // ✅ Correct
cartMutations.value.setCartDetail(data);  // ❌ Wrong
```

### strategyMutations Is Computed

`strategyMutations` returned by `useTransactionStore` **requires `.value`** when calling:

```javascript
const m = strategyMutations.value;
m.setPickupCode && m.setPickupCode('ABC123');
```

## Type Definitions

| File | Content |
|------|---------|
| `types/cart-transaction-store.d.ts` | `CartState`, `CartMutations`, `UseCartStoreReturn` |
| `types/shop-transaction-store.d.ts` | `ShopState`, `ShopMutations`, `UseShopStoreReturn` |
| `types/cart.ts` | `CartDataResponse`, `Product` and other cart interface types |

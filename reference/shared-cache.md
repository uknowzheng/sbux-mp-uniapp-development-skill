# SharedCache Cross-Page Shared Cache

## Use Cases

| Scenario | Description |
|----------|-------------|
| Cross-page data passing | Page A selects a store, Page B uses it |
| Temporary data caching | API data reused across pages, avoid duplicate requests |
| Flow state sharing | Order flow shares order draft across multiple pages |
| Non-reactive data | Pure data storage, no need to trigger view updates |

**Compared to other solutions**: Lighter than Vuex (pure data cache), faster than Storage (no serialization), more structured than globalData (modular + watch).

## Core Features

- Modular definition, API design similar to Pinia
- Global singleton, same data shared across pages
- Supports watch for monitoring data changes
- Pure JS implementation, no Vue reactivity dependency
- Supports direct assignment to trigger watch (Proxy implementation)

## Usage

### 1. Define Cache Module

Create module files under `common/sharedCache/modules/`:

```javascript
// common/sharedCache/modules/order.js
import { defineCache } from 'common/sharedCache';

export const useOrderCache = defineCache('order', () => ({
    currentOrder: null,
    selectedStore: null,
    cartItems: [],
}));
```

### 2. Read/Write Cache Data

```javascript
import { useOrderCache } from 'common/sharedCache/modules/order';

const orderCache = useOrderCache();

// Write (direct assignment)
orderCache.currentOrder = { id: 123, status: 'pending' };

// Read
console.log(orderCache.currentOrder);
```

### 3. Use Hook Operations

```javascript
import { useCacheActions } from 'common/sharedCache';
import { useOrderCache } from 'common/sharedCache/modules/order';

const { watch, watchAll, patch, reset, getState } = useCacheActions(useOrderCache);

// Watch single field
const unwatch = watch('currentOrder', (newVal, oldVal) => {
    console.log('Order changed:', newVal);
});

// Watch all fields
const unwatchAll = watchAll((key, newVal, oldVal) => {
    console.log(`${key} changed:`, newVal);
});

// Batch update
patch({ currentOrder: null, selectedStore: null });

// Functional batch update
patch(state => { state.cartItems.push({ sku: 'SKU001' }); });

// Reset to initial state
reset();

// Get current state snapshot
const snapshot = getState();

// Unwatch (call on page unload)
unwatch();
unwatchAll();
```

## Typical Scenarios

### Store Selection Cross-Page Sharing

```javascript
// common/sharedCache/modules/store.js
export const useStoreCache = defineCache('store', () => ({
    selectedStore: null,
    recentStores: [],
}));

// Store selection page
const storeCache = useStoreCache();
storeCache.selectedStore = { storeId: 'S001', name: 'Starbucks Store' };

// Order page (automatically gets selected store)
const storeCache = useStoreCache();
console.log(storeCache.selectedStore);
```

### API Data Caching (With Expiration)

```javascript
// common/sharedCache/modules/menu.js
export const useMenuCache = defineCache('menu', () => ({
    menuList: null,
    lastFetchTime: 0,
}));

async function getMenuWithCache() {
    const menuCache = useMenuCache();
    const now = Date.now();
    if (menuCache.menuList && now - menuCache.lastFetchTime < 5 * 60 * 1000) {
        return menuCache.menuList;
    }
    const data = await menuService.getMenuList();
    menuCache.menuList = data;
    menuCache.lastFetchTime = now;
    return data;
}
```

## API Reference

### defineCache(id, stateFactory)

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | Cache module unique identifier |
| stateFactory | () => object | Factory function returning initial state |

Returns: `useCache` function, calling it returns a cache instance.

### useCacheActions(useCacheFn)

| Method | Description |
|--------|-------------|
| `watch(key, callback)` | Watch single field, returns unwatch function |
| `watchAll(callback)` | Watch all fields, returns unwatch function |
| `patch(partialState)` | Batch update, supports object or function |
| `reset()` | Reset to initial state |
| `getState()` | Get current state snapshot |

### Global Methods

| Method | Description |
|--------|-------------|
| `getAllCaches()` | Get all cache modules |
| `clearAllCaches()` | Clear all caches (reset to initial state) |
| `destroyCache(id)` | Destroy specified cache module |

## Notes

1. **Unwatch promptly**: Call `unwatch()` on page `onUnload` or `onHide` to avoid memory leaks
2. **Avoid storing large objects**: Cache data stays in memory, avoid storing overly large data
3. **Unique module naming**: `defineCache` id must be globally unique
4. **Not persistent**: Cache clears on mini-program restart, use Storage for persistence

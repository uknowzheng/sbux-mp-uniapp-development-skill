# SharedCache 跨页面共享缓存

## 适用场景

| 场景 | 说明 |
|------|------|
| 跨页面数据传递 | A 页面选择门店，B 页面使用 |
| 临时数据缓存 | 接口数据多页面复用，避免重复请求 |
| 流程状态共享 | 下单流程多页面共享订单草稿 |
| 非响应式数据 | 纯数据存储，不需触发视图更新 |

**对比其他方案**：比 Vuex 轻量（纯数据缓存）、比 Storage 快（无序列化）、比 globalData 规范（模块化 + watch）。

## 核心特性

- 模块化定义，类似 Pinia 的 API 设计
- 全局单例，跨页面共享同一份数据
- 支持 watch 监听数据变化
- 纯 JS 实现，不依赖 Vue 响应式系统
- 支持直接赋值触发 watch（Proxy 实现）

## 使用方式

### 1. 定义缓存模块

在 `common/sharedCache/modules/` 下创建模块文件：

```javascript
// common/sharedCache/modules/order.js
import { defineCache } from 'common/sharedCache';

export const useOrderCache = defineCache('order', () => ({
    currentOrder: null,
    selectedStore: null,
    cartItems: [],
}));
```

### 2. 读写缓存数据

```javascript
import { useOrderCache } from 'common/sharedCache/modules/order';

const orderCache = useOrderCache();

// 写入（直接赋值）
orderCache.currentOrder = { id: 123, status: 'pending' };

// 读取
console.log(orderCache.currentOrder);
```

### 3. 使用 Hook 操作

```javascript
import { useCacheActions } from 'common/sharedCache';
import { useOrderCache } from 'common/sharedCache/modules/order';

const { watch, watchAll, patch, reset, getState } = useCacheActions(useOrderCache);

// 监听单个字段
const unwatch = watch('currentOrder', (newVal, oldVal) => {
    console.log('订单变化:', newVal);
});

// 监听所有字段
const unwatchAll = watchAll((key, newVal, oldVal) => {
    console.log(`${key} 变化:`, newVal);
});

// 批量更新
patch({ currentOrder: null, selectedStore: null });

// 函数式批量更新
patch(state => { state.cartItems.push({ sku: 'SKU001' }); });

// 重置为初始状态
reset();

// 获取当前状态快照
const snapshot = getState();

// 取消监听（页面卸载时调用）
unwatch();
unwatchAll();
```

## 典型场景

### 门店选择跨页面共享

```javascript
// common/sharedCache/modules/store.js
export const useStoreCache = defineCache('store', () => ({
    selectedStore: null,
    recentStores: [],
}));

// 门店选择页
const storeCache = useStoreCache();
storeCache.selectedStore = { storeId: 'S001', name: '星巴克门店' };

// 下单页（自动获取已选门店）
const storeCache = useStoreCache();
console.log(storeCache.selectedStore);
```

### 接口数据缓存（带过期时间）

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

## API 参考

### defineCache(id, stateFactory)

| 参数 | 类型 | 说明 |
|------|------|------|
| id | string | 缓存模块唯一标识 |
| stateFactory | () => object | 返回初始状态的工厂函数 |

返回：`useCache` 函数，调用后返回缓存实例。

### useCacheActions(useCacheFn)

| 方法 | 说明 |
|------|------|
| `watch(key, callback)` | 监听单个字段，返回取消监听函数 |
| `watchAll(callback)` | 监听所有字段，返回取消监听函数 |
| `patch(partialState)` | 批量更新，支持对象或函数 |
| `reset()` | 重置为初始状态 |
| `getState()` | 获取当前状态快照 |

### 全局方法

| 方法 | 说明 |
|------|------|
| `getAllCaches()` | 获取所有缓存模块 |
| `clearAllCaches()` | 清空所有缓存（重置为初始状态） |
| `destroyCache(id)` | 销毁指定缓存模块 |

## 注意事项

1. **及时取消监听**：在页面 `onUnload` 或 `onHide` 时调用 `unwatch()` 避免内存泄漏
2. **避免存储大对象**：缓存数据常驻内存，避免存储过大的数据
3. **模块命名唯一**：`defineCache` 的 id 必须全局唯一
4. **非持久化**：小程序重启后缓存会清空，需要持久化请配合 Storage 使用

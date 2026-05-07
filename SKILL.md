---
name: sbux-mp-uniapp-development-skill
description: UniApp 微信小程序项目完整开发指南，涵盖技术栈、命名规范、目录结构、页面/组件/composable 开发、路由导航、服务层、状态管理、埋点、监控上报、构建部署。Use when developing pages, components, composables, services, stores, or any feature in this UniApp mini-program project.
---

# UniApp 小程序开发指南

## Quick Command Reference

根据开发场景，按需加载对应的 command workflow 或 reference 文件：

| 命令 | 场景 / 意图 | 读取文件 |
|------|-------------|---------|
| `/page` | 新建页面、页面 Vue 写法、生命周期管理 | [commands/page.md](commands/page.md) |
| `/component` | 新建组件、弹窗组件、复杂组件拆分 | [commands/component.md](commands/component.md) |
| `/composable` | 编写主/子 composable、watch 管理 | [commands/composable.md](commands/composable.md) |
| `/service` | 新建服务域、API 对接、请求配置 | [commands/service.md](commands/service.md) |
| `/store` | 定义/使用 Store、defineStore | [commands/store.md](commands/store.md) |
| `/deploy` | 构建部署、CI/CD、环境变量 | [commands/deploy.md](commands/deploy.md) |
| `/monitor` | 监控上报、听云 SDK、异常捕获 | [commands/monitor.md](commands/monitor.md) |
| `/use-sbux-mp` | 端到端业务功能开发（全量规范） | [commands/use-sbux-mp.md](commands/use-sbux-mp.md) |

**详细参考文档**（command workflow 中的深入内容）：

| 主题 | 读取文件 |
|------|---------|
| Composable 完整架构、watch 规范 | [reference/composables.md](reference/composables.md) |
| 组件完整写法、弹窗/容器组件 | [reference/component.md](reference/component.md) |
| 服务层目录、请求配置、多渠道分流 | [reference/services.md](reference/services.md) |
| Store 完整用法、读写示例 | [reference/store.md](reference/store.md) |
| Transaction Store（购物车/门店/交易聚合层） | [reference/transaction-store.md](reference/transaction-store.md) |
| SharedCache 跨页面共享缓存 | [reference/shared-cache.md](reference/shared-cache.md) |
| 弹窗 Composables（Alert/Slide/Prompt） | [reference/dialog.md](reference/dialog.md) |
| 静态资源管理、imgUrl 用法 | [reference/static-assets.md](reference/static-assets.md) |
| CI/CD 流水线、GitHub Actions | [reference/deployment.md](reference/deployment.md) |
| 听云 SDK API、业务监控 Hook | [reference/monitor.md](reference/monitor.md) |
| 完整代码示例 | [examples.md](examples.md) |

**使用方式**: 
- 用户明确提到具体开发任务时，先读取对应 `commands/xxx.md` 获取 step-by-step workflow
- 需要更详细的规范时，再读取对应 `reference/xxx.md`
- 无明确场景时，SKILL.md 中的摘要足够指导

---

## 0. 技术栈

| 类别 | 技术 |
|------|------|
| 框架 | UniApp 2.0.2 + Vue 2.6.10 + Composition API (@vue/composition-api) |
| 构建 | Vue CLI 4.x + Webpack |
| 包管理 | pnpm + Node.js 18 |
| 代码规范 | ESLint 6.8.0 + Prettier 2.8.8 + Husky pre-commit |
| 状态管理 | Vuex 3.2.0 + Pinia 2.1.7 |
| 测试 | Jest + @vue/test-utils + @vue/cli-plugin-unit-jest |
| 样式 | Less |
| 小程序生命周期 | @dcloudio/uni-app |
| 环境变量 | `UNI_PLATFORM`（平台）、`NODE_CUSTOM_ENV`（环境） |
| 内存优化 | `NODE_OPTIONS="--max-old-space-size=4096"` |

---

## 1. 命名规范

所有文件和目录使用 **camelCase**，Hook 使用 **use + PascalCase**。

| 类型 | 规范 | 示例 |
|------|------|------|
| 目录 | camelCase | `userInfo/`, `orderDetail/` |
| 页面 | camelCase | `orderDetail.vue` |
| Hook | use + PascalCase | `useOrderDetail.js` |
| Service | camelCase | `orderService.js` |
| Store | camelCase | `useMainStore.js` |
| 组件目录 | camelCase | `productCard/index.vue` |
| 平台差异文件 | name.platform.js | `payment.weixin.js` |
| 埋点文件 | camelCase | `srkitDetail.js` |

### Hook 命名细分

- 页面 Hook: `use[PageName].js`
- 组件 Hook: `use[ComponentName].js`
- 通用 Hook: `use[FeatureDescription].js`

### 平台差异文件

- 微信: `xxx.weixin.js` | 支付宝: `xxx.alipay.js` | 抖音: `xxx.douyin.js` | H5: `xxx.h5.js`

### Store 命名

- 业务 Store: `use[BusinessName]Store.js`（如 `useMainStore.js`、`useOrderStore.js`）
- Mutation 常量: `UPDATE_[ACTION_NAME]`（仅内部使用，不导出）

---

## 2. 目录结构

### 页面目录

```bash
pages/
└── pay/                       # 页面目录
    ├── index.vue              # 页面组件
    ├── usePay.js              # 页面主 composable（管理生命周期）
    └── composables/           # 页面子 composables（纯逻辑，无生命周期）
        ├── useOrder.js
        ├── usePayMethod.js
        └── usePayTrack.js
```

### 旧页重构

在同级创建同名目录，逐步将逻辑提取到新目录，旧 `.vue` 文件保持不变：

```bash
pages/
├── orderDetail.vue              # 旧页面（保持不变）
└── orderDetail/                 # 新目录
    ├── composables/
    └── useOrderDetail.js
```

### 组件目录

```bash
components/
├── cardItem/                    # 简单组件 → index.vue
├── orderDetailPanel/           # 复杂组件 → index.vue + useOrderDetailPanel.js
└── svcPay/                      # 组件组 → payMent.vue, payPop.vue, paySuccess.vue
```

**组织原则**:
- 所有组件用目录形式，主组件命名 `index.vue`
- 简单组件：仅 `index.vue`
- 复杂组件：`index.vue` + `use[ComponentName].js`
- 组件组：多个关联子组件同目录，各自独立命名
- 全局通用组件在 `common/components/`，分包组件在分包的 `components/`

### 服务层目录

```bash
src/services/
├── index.js              # 统一导出入口
├── user/                 # 用户域
│   ├── index.js          # 服务方法
│   ├── api.js            # API 接口定义
│   ├── user.weixin.js    # 微信平台实现
│   └── user.h5.js        # H5 平台实现
├── order/
├── payment/              # 多渠道场景
└── map/                  # 多渠道地图
```

### 埋点目录

```bash
[packageName]/track/
├── index.js              # 导出入口
├── srkitList.js          # 按页面拆分
└── srkitDetail.js
```

---

## 3. 页面开发

### Vue 文件写法

**`<script setup>`（推荐，无平台差异时）**:

```vue
<template>
  <view class="order-detail">
    <order-header :info="orderInfo" />
  </view>
</template>

<script setup>
import { useOrderDetail } from './useOrderDetail';
const { orderInfo, fetchOrder } = useOrderDetail();
</script>
```

**Options `setup()`（有 `#ifdef` 条件编译时必须）**:

```vue
<script>
import { useOrderDetail } from './useOrderDetail';
// #ifdef H5
import H5Navigation from '../components/h5Header/index.vue';
// #endif

export default {
  components: {
    // #ifdef H5
    H5Navigation,
    // #endif
  },
  setup() {
    return useOrderDetail();
  }
};
</script>
```

### 生命周期管理

**核心原则：主 composable 管理所有生命周期，子 composables 不包含生命周期。**

```javascript
// usePay.js - 主 composable
import { onLoad, onShow, onUnload, onPageScroll } from '@dcloudio/uni-app';

export function usePay() {
  onLoad((params) => { initPage(params); });
  onShow(() => { trackPageView(); });
  onPageScroll((e) => { handlePageScroll(e); });
  onUnload(() => { cleanup(); });

  // 引入子 composables（纯逻辑，无生命周期）
  const { orderReviewData } = useOrder();
  const { selectedPayMethod } = usePayMethod();

  // 跨 composables 的 watch 只在主 composable 中
  watch(selectedPayMethod, (newVal) => { trackPaymentToolAction(newVal); });

  return { orderReviewData, selectedPayMethod };
}
```

### 需手动声明的生命周期

Vue 2 + @vue/composition-api 环境中，部分钩子需在 `.vue` 文件显式声明空方法：

| 生命周期 | 是否需要手动声明 | 说明 |
|---------|----------------|------|
| `onLoad` | 否 | 自动注入 |
| `onShow` | 否 | 自动注入 |
| `onReady` | 否 | 自动注入 |
| `onHide` | 否 | Babel 自动注入 |
| `onUnload` | 否 | Babel 自动注入 |
| `onReachBottom` | **是** | 触底加载 |
| `onShareAppMessage` | **是** | 分享功能 |
| `onPullDownRefresh` | **是** | 下拉刷新 |
| `onPageScroll` | **是** | 页面滚动 |

```javascript
// Options setup() 中声明
export default {
  onReachBottom() {},
  onShareAppMessage() {},
  onPullDownRefresh() {},
  onPageScroll() {},
  setup() { return usePay(); }
};
```

---

## 4. Composable 开发要点

> 完整架构与示例详见 [reference/composables.md](reference/composables.md)

### 主/子分层原则

| 层级 | 位置 | 职责 | 生命周期 |
|------|------|------|---------|
| 主 composable | `pages/[pageName]/use[PageName].js` | 管理生命周期 + 协调子 composables | 有 |
| 子 composable - 数据层 | `composables/use[Feature]Data.js` | 数据获取和状态管理 | 无 |
| 子 composable - 业务层 | `composables/use[Feature]Logic.js` | 业务逻辑处理 | 无 |
| 子 composable - 工具层 | `composables/use[Feature].js` | 工具函数封装 | 无 |
| 子 composable - 埋点层 | `composables/use[Feature]Track.js` | 埋点逻辑 | 无 |
| 分包共享 | `[package]/composables/` | 分包多页面共享逻辑 | 无 |
| 全局共享 | `common/composables/` | 跨分包通用逻辑 | 无 |

### 关键约束

1. **子 composable 严禁包含生命周期**（onLoad, onShow 等）
2. **跨 composables 的 watch 只在主 composable 中管理**
3. **子 composable 可依赖其他子 composables**，形成依赖树
4. **命名导出**: `export function useXxx() {}`，禁止 default export

---

## 5. 组件开发要点

> 完整写法与示例详见 [reference/component.md](reference/component.md)

### 组件分类

| 类型 | 说明 | 示例 |
|------|------|------|
| 业务逻辑组件 | 按功能模块拆分 | paymentList, orderType, address, submit |
| 交互组件 | 弹窗/浮层 | SVCCardListPopup, PayMethodPopup, CompleteAddressDialog |
| 容器组件 | 布局和样式容器 | addressAndOrderContain, scrollFloatButton |
| 选项行组件 | 表单项变体 | optionRow/priceRow.vue, radioRow.vue, selectRow.vue |

### 最佳实践

1. **Props 传递**：复杂页面用 props 传递数据，避免过度依赖全局状态
2. **事件通信**：`$emit` 向父组件传递事件，保持单向数据流
3. **弹窗组件**：支持 open/close 方法，通过 ref 调用
4. **图片兜底**：使用 `imgUrl` 函数处理图片路径
5. **样式隔离**：使用 scoped 样式

---

## 6. 路由导航

使用 `common/router.config` 中的路由常量，避免硬编码路径。

### 路由常量

| 常量 | 描述 | 示例 |
|------|------|------|
| MOD | MOD 分包路由 | `MOD.menu`, `MOD.orderDetail` |
| MOP | MOP 分包路由 | `MOP.menu`, `MOP.cart` |
| EC | EC 分包路由 | `EC.ecHome` |
| USER | 用户分包路由 | `USER.memberCode`, `USER.login` |

### 导航方法

```javascript
import { navigateTo, redirectTo, switchTab, webviewTo } from 'common/utils/router.js';
import { MOD, MOP, USER } from 'common/router.config';

navigateTo({ url: MOD.menu });
navigateTo({ url: USER.memberCode });
navigateTo({ url: MOP.orderDetail, params: { orderId: '123' } });
webviewTo(termInfo.url);

// H5 环境 DeepLink
// #ifdef H5
import { DeepLinkNavigate } from '@sbux/sbuxjs-jsbridge';
DeepLinkNavigate(`sbuxcn://h5-webview?url=${encodeURIComponent(url)}`);
// #endif
```

**禁止**: 硬编码路径 `navigateTo({ url: '/shares/mod/pages/menu' })`

---

## 7. 服务层要点

> 完整指南详见 [reference/services.md](reference/services.md)

### 职责边界

**只负责**: 接口请求 + 简单数据转换
**禁止**: 路由导航逻辑、复杂业务逻辑（应在页面 hooks 中）

### 使用方式

```javascript
// 统一入口导入（推荐）
import { userService, orderService } from 'services';

// 域级导入
import { getUserInfo } from 'services/user';
```

### 新建域服务步骤

1. 创建域目录 `src/services/[domain]/`
2. 创建 `api.js` 定义接口
3. 创建 `index.js` 导出服务函数
4. 在 `src/services/index.js` 中注册: `export * as productService from './product'`

---

## 8. Store 使用要点

> 完整用法详见 [reference/store.md](reference/store.md)

当前为过渡阶段，Store 数据仍来自 Vuex，仅语法扩展。

### 快速上手

```javascript
// 定义
import { defineStore } from './defineStore.js';
export const useMainStore = defineStore('main', ({ state, commit }) => {
  const riskInfo = computed(() => state.riskInfo.value);
  const setRiskInfo = (data) => commit('UPDATE_MAIN_RISKINFO', data);
  return { riskInfo, setRiskInfo };
});

// 使用
const mainStore = useMainStore();
mainStore.riskInfo.value;          // JS 中读取（需要 .value）
mainStore.setRiskInfo({ data });   // 修改（禁止直接赋值 .value）
// <template> 中 {{ riskInfo }}    // 模板自动解包，无需 .value
```

### 关键约束

| 场景 | 写法 |
|------|------|
| JS 中读值 | `mainStore.riskInfo.value` |
| 模板中读值 | `{{ riskInfo }}`（自动解包） |
| 修改状态 | `mainStore.setRiskInfo({ data: xxx })` |
| Watch | `watch(() => mainStore.riskInfo.value, ...)` |
| **禁止** | `mainStore.riskInfo.value = xxx`（直接赋值） |

---

## 8.5. SharedCache 快速上手

> 完整 API 详见 [reference/shared-cache.md](reference/shared-cache.md)

跨页面共享缓存的轻量级方案（比 Vuex 轻量、比 Storage 快）。

```javascript
import { defineCache } from 'common/sharedCache';
import { useCacheActions } from 'common/sharedCache';

// 定义
export const useOrderCache = defineCache('order', () => ({ currentOrder: null, cartItems: [] }));

// 读写
const orderCache = useOrderCache();
orderCache.currentOrder = { id: 123 };  // 直接赋值
console.log(orderCache.currentOrder);    // 读取

// watch + patch
const { watch, patch, reset } = useCacheActions(useOrderCache);
const unwatch = watch('currentOrder', (newVal) => { /* ... */ });
patch({ currentOrder: null });
reset();
```

**注意**：`onUnload` 时调用 `unwatch()` 避免内存泄漏；缓存非持久化，重启后清空。

---

## 8.6. 弹窗 Composables

> 完整 API 详见 [reference/dialog.md](reference/dialog.md)

| Composable | 组件 | 用途 |
|-----------|------|------|
| `useAlertDialog` | `uni-alert` | 确认/取消类弹窗（支持 Promise） |
| `useSlideDialog` | `slide-dialog` | 信息展示类弹窗（支持富文本） |
| `usePromptDialog` | `uni-prompt` | 多按钮选择类弹窗 |

```javascript
import { useAlertDialog } from 'common/composables/useAlertDialog.js';
const { alertState, showAlert, handleAlertConfirm, handleAlertCancel } = useAlertDialog();

// 回调模式
showAlert({ title: '删除确认', onConfirm: () => { /* ... */ } });

// Promise 模式
const confirmed = await showAlert({ title: '确认操作', cancelEnable: true });
```

---

## 8.7. Transaction Store

> 完整 API 详见 [reference/transaction-store.md](reference/transaction-store.md)

交易核心 Store，从 `common/stores/useTransactionStore.js` 统一导入：

| Store | 职责 |
|-------|------|
| `useCartStore` | 购物车状态（商品列表、优惠券） |
| `useShopStore` | 门店状态（当前门店、预约时间） |
| `useTransactionStore` | 聚合层（baseOrder、strategy） |

```javascript
import { useCartStore, useShopStore } from 'common/stores/useTransactionStore.js';
const { shoppingCartDetail, cartMutations } = useCartStore(channel);
const { currentStore, shopMutations } = useShopStore(channel);
```

---

## 8.8. 静态资源

> 完整规范详见 [reference/static-assets.md](reference/static-assets.md)

图片引用必须使用 `imgUrl()` 方法：

```vue
<image :src="imgUrl('shares/public/images/mod/banner.png')" />
<image src="imgUrl('shares/common/icon.png')" />
```

相对路径自动添加 `/static/` 前缀，完整 URL 保持不变。仅支持 `imgUrl()`，不支持 `$imgUrl()`。

---

## 9. 埋点开发要点

### 命名规范

| 埋点类型 | 命名格式 | 示例 |
|---------|---------|------|
| 页面浏览 | `trackPageView` | `trackPageView()` |
| 点击事件 | `track[Element]Click` | `trackCouponClick()` |
| 曝光事件 | `track[Element]Expose` | `trackBannerExpose()` |
| 提交事件 | `track[Form]Submit` | `trackOrderSubmit()` |

### 设计原则

1. **封装成函数**：埋点代码放在 `track/` 目录，不在业务逻辑中散落
2. **按页面拆分**：每个页面独立 track 文件
3. **处理平台差异**：`#ifdef` 在 track 函数内部处理
4. **在 Hook 中调用**：业务 Hook 引入 track 函数，在适当时机调用

```javascript
// track/srkitDetail.js
export function trackCouponClick(detailData, extData, title) {
  // #ifdef MP-WEIXIN
  newSendTrackEx('SRKIT_DETAIL_CLICK', 'SRKIT_DETAIL_PAGE', { ... }, true);
  // #endif
}

// composables/useSrkitDetail.js
import { trackCouponClick } from '../track/srkitDetail';
const handleCouponClick = () => {
  trackCouponClick(detailData.value, extData.value, '优惠券');
};
```

---

## 10. 监控快速上手

> 完整 API 与封装详见 [reference/monitor.md](reference/monitor.md)

```javascript
import { useMonitorReport } from 'tingyun/index.js';

const monitorReport = useMonitorReport();

// 分级上报
monitorReport.error({ name: 'OrderSubmitError', message: '订单提交失败', stack: error.stack });
monitorReport.info({ name: 'OrderSubmitStart', message: '开始提交订单' });
monitorReport.warn({ name: 'LowBalance', message: '余额不足' });
```

**日志级别**: error > warn > info > debug > log

**注意事项**: 上报不阻塞业务、高频操作需节流、不上报敏感信息

---

## 11. 构建与运行

> 完整 CI/CD 配置详见 [reference/deployment.md](reference/deployment.md)

### 常用构建命令

```bash
# 微信小程序
pnpm run app:dev:mp-weixin       # 开发
pnpm run app:stg:mp-weixin       # 测试
pnpm run app:prod:mp-weixin      # 生产

# 支付宝小程序
pnpm run app:dev:mp-alipay

# H5 业务线
pnpm run dev:h5-module-config
pnpm run stg:h5-module-config

# 定制化（离线包）
pnpm run dev:h5-customization-mod
pnpm run build:customization-mod
```

### 构建环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `NODE_CUSTOM_ENV` | 环境 | dev / stg / prod |
| `UNI_PLATFORM` | 平台 | mp-weixin / mp-alipay / h5-xxx |
| `BUILD_TYPE` | 构建类型 | OfflinePackage（可选） |
| `CURR_MACHINE` | 设备 | android / ios（离线包） |
| `BUSINESS_TYPE` | 业务类型 | MOD / MOP（离线包） |

---

## Additional Resources

- Composable 完整架构 → [reference/composables.md](reference/composables.md)
- 组件完整写法 → [reference/component.md](reference/component.md)
- 服务层完整指南 → [reference/services.md](reference/services.md)
- Store 完整用法 → [reference/store.md](reference/store.md)
- Transaction Store → [reference/transaction-store.md](reference/transaction-store.md)
- SharedCache 共享缓存 → [reference/shared-cache.md](reference/shared-cache.md)
- 弹窗 Composables → [reference/dialog.md](reference/dialog.md)
- 静态资源管理 → [reference/static-assets.md](reference/static-assets.md)
- 构建部署完整配置 → [reference/deployment.md](reference/deployment.md)
- 监控完整 API → [reference/monitor.md](reference/monitor.md)
- 完整代码示例 → [examples.md](examples.md)

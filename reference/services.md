# Service Layer Full Development Guide

## Responsibility Boundaries

**Service Layer is only responsible for**:
- API requests
- Simple data transformation

**Should not contain**:
- Route navigation logic
- Complex business logic (should be in page hooks)

**Coding Standard**: Use functional style consistently, avoid class style

## Directory Structure

```bash
src/services/
├── index.js              # Unified export entry
├── user/                 # User domain
│   ├── index.js          # Service methods (routing entry)
│   ├── api.js            # API definitions
│   ├── user.weixin.js    # WeChat platform implementation
│   ├── user.alipay.js    # Alipay platform implementation
│   └── user.h5.js        # H5 platform implementation
├── order/                # Order domain
├── shop/                 # Store domain
├── coupon/               # Coupon domain
├── payment/              # Payment domain (multi-channel scenario)
│   ├── index.js
│   ├── payment.weixin.js
│   ├── payment.alipay.js
│   ├── payment.douyin.js
│   └── payment.h5.js
└── map/                  # Map domain (multi-channel)
    ├── index.js
    ├── map.weixin.js
    ├── map.amap.js
    └── map.h5.js
```

## Steps to Create New Domain Service

### 1. Create Domain Directory

Create a new directory under `src/services/`, e.g., `product/`

### 2. Create api.js

```javascript
/**
 * Product domain API definitions
 */
import { getApi } from 'common/utils/api.js';

const apis = {
  getProductList: '/product/list',
  getProductDetail: '/product/detail',
  searchProducts: '/product/search',
};

export default getApi(apis);
```

### 3. Create index.js (Functional Style)

```javascript
/**
 * Product domain service
 */
import http from 'common/utils/http/http.js';
import api from './api';

/**
 * Get product list
 * @param {Object} params - Query parameters
 * @param {Object} options - Request configuration
 */
export async function getProductList(params = {}, options = {}) {
  const data = await http.get(api.getProductList, params, true, options);
  return {
    list: data.list || [],
    total: data.total || 0,
  };
}

/**
 * Get product detail
 * @param {string} productId - Product ID
 */
export async function getProductDetail(productId, options = {}) {
  return http.get(api.getProductDetail, { productId }, true, options);
}

/**
 * Search products
 * @param {string} keyword - Search keyword
 * @param {Object} params - Other parameters
 */
export async function searchProducts(keyword, params = {}, options = {}) {
  return http.get(api.searchProducts, { keyword, ...params }, true, options);
}
```

### 4. Register to Unified Entry

```javascript
// src/services/index.js
export * as productService from './product';
```

## Domain Division Principles

### Divide by Business Boundary

- User-related: login, registration, personal info, address management → `user/`
- Order-related: place order, order list, order detail → `order/`
- Store-related: store search, favorite stores → `shop/`
- Coupon-related: claim, use, list → `coupon/`
- Payment-related: payment processing, payment methods → `payment/`
- Map-related: map search, map display → `map/`

### Anti-Patterns

- Don't divide by technical type (e.g., `get/`, `post/`)
- Don't divide by page (e.g., `homePage/`, `minePage/`)
- Don't create overly fine-grained domains

## Relationship with Existing Code

| Directory | Responsibility |
|-----------|---------------|
| `common/services/` | Cross-domain common services: auth, SDK, system |
| `customized/services/` | Platform-specific service implementations |
| `shares/services/` | In-package business services, recommended to gradually migrate to `services/` for unified management |

## Request Configuration Options

```javascript
{
  usePreventTool: true,    // Enable request debounce
  isCache: true,           // Cache response data
  forceCache: true,        // Force use cache
  isNoToast: true,         // Don't show error toast
  isErrorCompensate: true, // Auto retry on error
  timeout: 10000,          // Timeout (ms)
  deviceInfo: true,        // Include device info
}
```

## Common Patterns

### Data Fetching Pattern

```javascript
export async function fetchData(id, options = {}) {
  const data = await http.get(api.getDetail, { id }, true, options);
  return data;
}
```

### List Fetching Pattern

```javascript
export async function fetchList(params = {}, options = {}) {
  const data = await http.get(api.getList, params, true, options);
  return {
    list: data.list || [],
    total: data.total || 0,
    hasMore: data.hasMore || false,
  };
}
```

### Error Handling Pattern

```javascript
export async function updateData(data, options = {}) {
  try {
    const res = await http.post(api.update, data, true, options);
    return res;
  } catch (error) {
    uni.showToast({ title: error.message, icon: 'none' });
    throw error;
  }
}
```

## Best Practices

1. **Functional style**: Use functional style consistently, avoid class
2. **Single responsibility**: Only handle API requests and simple data transformation
3. **No routing**: Don't handle route navigation in service layer
4. **Business separation**: Complex business logic goes in page hooks
5. **Error handling**: Use try-catch-finally
6. **Data transformation**: Keep simple and predictable

## Multi-Channel Service Design

When services need different implementations for different platforms (WeChat/Alipay/Douyin/H5, etc.), use the routing pattern.

### Routing Entry Pattern

```javascript
// services/payment/index.js
// #ifdef MP-WEIXIN
export { default } from './payment.weixin.js';
export * from './payment.weixin.js';
// #endif

// #ifdef MP-ALIPAY
export { default } from './payment.alipay.js';
export * from './payment.alipay.js';
// #endif

// #ifdef MP-TOUTIAO
export { default } from './payment.douyin.js';
export * from './payment.douyin.js';
// #endif

// #ifdef H5
// #ifndef H5-CMB
export { default } from './payment.h5.js';
export * from './payment.h5.js';
// #endif
// #endif
```

### Custom Conditional Compilation Macros

The project defines custom macros in `src/package.json`:

| Macro Name | Description | Based On |
|------------|-------------|----------|
| MP-DINGTALK | DingTalk mini-program | mp-alipay |
| H5-CMB | CMB H5 | h5 |

To add a new custom macro, add it in `src/package.json`:

```json
{
    "uni-app": {
        "scripts": {
            "h5-xxx": {
                "title": "XXX Channel H5",
                "env": { "UNI_PLATFORM": "h5" },
                "define": { "H5-XXX": true }
            }
        }
    }
}
```

### Response Model Mapping

When different channels return inconsistent data structures, define a unified response model:

```bash
services/user/
├── index.js              # Routing entry
├── api.js                # API definitions
├── model.js              # Unified response model definition
├── user.weixin.js        # WeChat implementation (with mapper)
└── user.h5.js            # H5 implementation (with mapper)
```

```javascript
// services/user/model.js
export const UserInfoModel = {
    userId: '',
    nickname: '',
    avatar: '',
    phone: '',
    memberLevel: '',      // green/gold/diamond
    starCount: 0,
};
```

```javascript
// services/user/user.weixin.js - With mapper
function mapUserInfo(raw) {
    return {
        userId: raw.user_id || raw.userId,
        nickname: raw.nick_name || '',
        avatar: raw.avatar_url || '',
        phone: raw.mobile || '',
        memberLevel: mapMemberLevel(raw.loyalty_tier),
        starCount: Number(raw.star_count || 0),
    };
}

export async function getUserInfo(options = {}) {
    const raw = await http.get(api.getUserInfo, {}, true, options);
    return mapUserInfo(raw);
}
```

For complex mapping logic, extract into separate mapper files:

```bash
services/order/
├── mapper/
│   ├── weixin.mapper.js
│   ├── alipay.mapper.js
│   └── h5.mapper.js
├── order.weixin.js
└── order.h5.js
```

### Multi-Channel Design Principles

1. **Consistent interface signatures**: Method names, parameters, and return values must be consistent across platform implementations
2. **Cohesive differences**: Platform differences are encapsulated within specific implementation files, not exposed to callers
3. **Simple routing entry**: index.js only does conditional compilation routing, no business logic
4. **Shared logic reuse**: Multi-platform shared logic extracted to api.js or separate utils files
5. **Unified response**: When channels return inconsistent data, convert to unified models via mapper

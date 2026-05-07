# 服务层完整开发指南

## 职责边界

**Service Layer 只负责**:
- 接口请求
- 简单数据转换

**不应包含**:
- 路由导航逻辑
- 复杂业务逻辑（应在页面 hooks 中）

**编码规范**: 统一使用函数式写法，避免 class 写法

## 目录结构

```bash
src/services/
├── index.js              # 统一导出入口
├── user/                 # 用户域
│   ├── index.js          # 服务方法（路由入口）
│   ├── api.js            # API 接口定义
│   ├── user.weixin.js    # 微信平台实现
│   ├── user.alipay.js    # 支付宝平台实现
│   └── user.h5.js        # H5 平台实现
├── order/                # 订单域
├── shop/                 # 门店域
├── coupon/               # 优惠券域
├── payment/              # 支付域（多渠道场景）
│   ├── index.js
│   ├── payment.weixin.js
│   ├── payment.alipay.js
│   ├── payment.douyin.js
│   └── payment.h5.js
└── map/                  # 地图域（多渠道）
    ├── index.js
    ├── map.weixin.js
    ├── map.amap.js
    └── map.h5.js
```

## 新建域服务步骤

### 1. 创建域目录

在 `src/services/` 下创建新目录，如 `product/`

### 2. 创建 api.js

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

### 3. 创建 index.js（函数式写法）

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

### 4. 注册到统一入口

```javascript
// src/services/index.js
export * as productService from './product';
```

## 域划分原则

### 按业务边界划分

- 用户相关: 登录、注册、个人信息、地址管理 → `user/`
- 订单相关: 下单、订单列表、订单详情 → `order/`
- 门店相关: 门店查询、收藏门店 → `shop/`
- 优惠券相关: 领取、使用、列表 → `coupon/`
- 支付相关: 支付处理、支付方式 → `payment/`
- 地图相关: 地图查询、地图显示 → `map/`

### 反模式

- 不要按技术类型划分（如 `get/`、`post/`）
- 不要按页面划分（如 `homePage/`、`minePage/`）
- 不要创建过度细粒度的域

## 与现有代码的关系

| 目录 | 职责 |
|------|------|
| `common/services/` | 跨域通用服务：认证、SDK、系统 |
| `customized/services/` | 平台定制服务实现 |
| `shares/services/` | 分包内业务服务，建议逐步迁移到 `services/` 统一管理 |

## 请求配置选项

```javascript
{
  usePreventTool: true,    // 启用请求防抖
  isCache: true,           // 缓存响应数据
  forceCache: true,        // 强制使用缓存
  isNoToast: true,         // 不显示错误 toast
  isErrorCompensate: true, // 错误自动重试
  timeout: 10000,          // 超时时间（ms）
  deviceInfo: true,        // 包含设备信息
}
```

## 常见模式

### 数据获取模式

```javascript
export async function fetchData(id, options = {}) {
  const data = await http.get(api.getDetail, { id }, true, options);
  return data;
}
```

### 列表获取模式

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

### 错误处理模式

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

## 最佳实践

1. **函数式写法**：统一使用函数式，避免 class
2. **单一职责**：只处理接口请求和简单数据转换
3. **不处理路由**：不在 service 层处理路由导航
4. **业务分离**：复杂业务逻辑放在页面 hooks 中
5. **错误处理**：使用 try-catch-finally
6. **数据转换**：保持简单和可预测

## 多渠道服务设计

当服务需要针对不同平台（微信/支付宝/抖音/H5 等）有差异化实现时，采用分流模式。

### 分流入口写法

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

### 自定义条件编译宏

项目在 `src/package.json` 中定义了自定义宏：

| 宏名称 | 说明 | 基于平台 |
|--------|------|----------|
| MP-DINGTALK | 钉钉小程序 | mp-alipay |
| H5-CMB | 招商银行H5 | h5 |

如需新增自定义宏，在 `src/package.json` 中添加：

```json
{
    "uni-app": {
        "scripts": {
            "h5-xxx": {
                "title": "XXX渠道H5",
                "env": { "UNI_PLATFORM": "h5" },
                "define": { "H5-XXX": true }
            }
        }
    }
}
```

### 响应体模型映射

当不同渠道返回数据结构不一致时，定义统一响应模型：

```bash
services/user/
├── index.js              # 分流入口
├── api.js                # API 定义
├── model.js              # 统一响应模型定义
├── user.weixin.js        # 微信实现（含 mapper）
└── user.h5.js            # H5 实现（含 mapper）
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
// services/user/user.weixin.js - 含 mapper
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

复杂映射逻辑可抽为独立 mapper 文件：

```bash
services/order/
├── mapper/
│   ├── weixin.mapper.js
│   ├── alipay.mapper.js
│   └── h5.mapper.js
├── order.weixin.js
└── order.h5.js
```

### 多渠道设计原则

1. **接口签名一致**：各平台实现的方法名、参数、返回值保持一致
2. **差异内聚**：平台差异封装在具体实现文件内，不暴露给调用方
3. **分流入口简洁**：index.js 只做条件编译分流，不包含业务逻辑
4. **公共逻辑复用**：多平台共用逻辑抽到 api.js 或单独 utils 文件
5. **响应体统一**：各渠道返回不一致时，通过 mapper 转换为统一模型

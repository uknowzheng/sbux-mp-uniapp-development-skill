# /service - 新建服务域

## 开发流程

### Step 1: 创建域目录

在 `src/services/` 下创建新目录，如 `product/`

### Step 2: 创建 api.js

使用 `getApi` 定义接口路径：

```javascript
import { getApi } from 'common/utils/api.js';
const apis = { getProductList: '/product/list', getProductDetail: '/product/detail' };
export default getApi(apis);
```

### Step 3: 创建 index.js（函数式写法）

导入 `http` + `api`，导出 async 函数。统一使用函数式写法，避免 class。

### Step 4: 注册到统一入口

```javascript
// src/services/index.js
export * as productService from './product';
```

### Step 5: 处理平台差异（如需要）

多渠道场景使用条件编译分流入口（`#ifdef`），各平台实现保持接口签名一致。需要时创建 `model.js` 定义统一响应模型 + mapper 函数。

## 关键约束

1. 只负责接口请求 + 简单数据转换
2. 禁止包含路由导航逻辑
3. 禁止包含复杂业务逻辑（应在页面 hooks 中）
4. 统一函数式写法，避免 class
5. 按业务边界划分域，不要按技术类型或页面划分

## 常用请求配置

`options` 参数：`usePreventTool`(防抖) / `isCache`(缓存) / `isNoToast`(无错误提示) / `timeout`(超时)

## 详细参考

→ 读取 [reference/services.md](../reference/services.md) 获取完整目录结构、请求配置、多渠道分流、响应体模型映射

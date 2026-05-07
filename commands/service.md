# /service - Create New Service Domain

## Development Workflow

### Step 1: Create Domain Directory

Create a new directory under `src/services/`, e.g., `product/`

### Step 2: Create api.js

Use `getApi` to define interface paths:

```javascript
import { getApi } from 'common/utils/api.js';
const apis = { getProductList: '/product/list', getProductDetail: '/product/detail' };
export default getApi(apis);
```

### Step 3: Create index.js (Functional Style)

Import `http` + `api`, export async functions. Use functional style consistently, avoid class.

### Step 4: Register to Unified Entry

```javascript
// src/services/index.js
export * as productService from './product';
```

### Step 5: Handle Platform Differences (If Needed)

For multi-channel scenarios, use conditional compilation routing entry (`#ifdef`). Keep interface signatures consistent across platform implementations. Create `model.js` for unified response models + mapper functions when needed.

## Key Constraints

1. Only responsible for API requests + simple data transformation
2. No route navigation logic
3. No complex business logic (should be in page hooks)
4. Use functional style consistently, avoid class
5. Divide domains by business boundary, not by technical type or page

## Common Request Options

`options` parameter: `usePreventTool`(debounce) / `isCache`(cache) / `isNoToast`(no error toast) / `timeout`(timeout)

## Detailed Reference

→ Read [reference/services.md](../reference/services.md) for full directory structure, request config, multi-channel routing, response model mapping

# /deploy - 构建与部署

## 开发流程

### Step 1: 选择构建命令

```bash
# 微信小程序
pnpm run app:dev:mp-weixin       # 开发
pnpm run app:stg:mp-weixin       # 测试
pnpm run app:prod:mp-weixin      # 生产

# 支付宝小程序
pnpm run app:dev:mp-alipay

# H5 业务线
pnpm run dev:h5-module-config    # 开发
pnpm run stg:h5-module-config    # 测试

# 定制化（离线包）
pnpm run dev:h5-customization-mod
pnpm run build:customization-mod # 生产
```

### Step 2: 设置环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `NODE_CUSTOM_ENV` | 环境 | dev / stg / prod |
| `UNI_PLATFORM` | 平台 | mp-weixin / mp-alipay / h5-xxx |
| `BUILD_TYPE` | 构建类型 | OfflinePackage（可选） |
| `BUSINESS_TYPE` | 业务类型 | MOD / MOP（离线包） |

### Step 3: 内存优化（开发时）

```bash
export NODE_OPTIONS="--max-old-space-size=4096"
```

### Step 4: 构建产出

构建产物输出到 `dist/` 目录。

## 构建流程说明

`build-script/platform-builder.js` 核心流程：
1. 读取环境变量 → 2. 执行平台构建脚本 → 3. 生成 pages.json/manifest.json → 4. 复制 share modules → 5. Vue CLI 构建 → 6. 输出到 dist/

## CI/CD 流水线

- **develop 分支** → 自动部署测试环境
- **main 分支** → 人工审批后部署生产环境
- **PR** → 自动 lint + test + 构建验证

## 分支策略

`main`(生产) → `develop`(测试) → `feature/*`(功能) → `hotfix/*`(紧急修复)

## 详细参考

→ 读取 [reference/deployment.md](../reference/deployment.md) 获取完整 CI/CD 配置、GitHub Actions YAML、部署脚本

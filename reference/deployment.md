# 构建部署完整指南

## 当前构建体系

### package.json Scripts 结构

```bash
# 多平台小程序构建
app:dev:mp-weixin      # 微信小程序开发
app:stg:mp-weixin      # 微信小程序测试
app:prod:mp-weixin     # 微信小程序生产
app:dev:mp-alipay      # 支付宝小程序
app:dev:mp-toutiao     # 抖音小程序

# H5 多业务线构建
dev:h5-module-config   # H5 模块配置开发
stg:h5-module-config   # H5 模块配置测试
dev:h5-generalActivity # H5 通用活动
dev:h5-loyalty         # H5 会员夜

# 电子券/票
dev:ele-coupon
stg:ele-coupon

# 定制化业务（离线包）
dev:h5-customization-mod
dev:h5-customization-mop
stg:customization-mod
build:customization-mod
```

### build-script 核心流程

`build-script/platform-builder.js` 核心逻辑：

1. 读取环境变量
   - `NODE_CUSTOM_ENV`: dev/stg/prod
   - `TARGET_PLATFORM`: mp-weixin/mp-alipay/h5-xxx
   - `BUILD_TYPE`: OfflinePackage（可选）
   - `CURR_MACHINE`: android/ios（离线包）
   - `BUSINESS_TYPE`: MOD/MOP（离线包）

2. 构建流程
   - 执行平台特定构建脚本
   - 生成 pages.json、manifest.json
   - 复制 share modules
   - 执行 Vue CLI 构建

3. 输出到 `dist/` 目录

## 环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `NODE_CUSTOM_ENV` | 环境标识 | dev / stg / prod |
| `UNI_PLATFORM` | 平台标识 | mp-weixin / mp-alipay / h5-xxx |
| `BUILD_TYPE` | 构建类型 | OfflinePackage（可选） |
| `CURR_MACHINE` | 设备类型 | android / ios（离线包） |
| `BUSINESS_TYPE` | 业务类型 | MOD / MOP（离线包） |

## CI/CD 流水线设计

### GitHub Actions 工作流

```yaml
# .github/workflows/deploy-pipeline.yml
name: UniApp Deploy Pipeline

on:
  push:
    branches: [develop, release/*, main]
  pull_request:
    branches: [develop, main]

jobs:
  # 阶段1: 代码质量检查
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install
      - name: Run ESLint
        run: pnpm run lint
      - name: Run Unit Tests
        run: pnpm vitest run --coverage

  # 阶段2: 构建并预览（PR时）
  build-preview:
    if: github.event_name == 'pull_request'
    needs: lint-and-test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        platform: [mp-weixin, h5-module-config]
        env: [dev]
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install
      - name: Build ${{ matrix.platform }}
        run: pnpm run app:${{ matrix.env }}:${{ matrix.platform }}

  # 阶段3: 部署到测试环境
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: lint-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install
      - name: Build WeChat Mini Program
        run: pnpm run app:stg:mp-weixin
      - name: Build H5
        run: pnpm run stg:h5-module-config

  # 阶段4: 部署到生产环境（需人工审批）
  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: lint-and-test
    environment:
      name: production
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install
      - name: Build Production Artifacts
        run: |
          pnpm run app:prod:mp-weixin
          pnpm run app:prod:mp-alipay
```

### 部署脚本示例

```bash
#!/bin/bash
# scripts/release-miniprogram.sh

echo "开始发布小程序..."

# 微信小程序
npx miniprogram-ci upload \
  --appid $WECHAT_APPID \
  --private-key $WECHAT_PRIVATE_KEY \
  --project-path dist/build/mp-weixin \
  --version $(node -p "require('./package.json').version") \
  --desc "Production Release"

# 支付宝小程序
npx minidev upload \
  --app-id $ALIPAY_APPID \
  --private-key $ALIPAY_PRIVATE_KEY \
  --project dist/build/mp-alipay \
  --version $(node -p "require('./package.json').version")

echo "发布完成！"
```

## 最佳实践

1. **分支策略**: main(生产) → develop(测试) → feature/*(功能) → hotfix/*(紧急修复)
2. **自动化检查**: 所有 PR 必须通过 CI、ESLint 零错误
3. **安全策略**: 生产部署需人工审批，Secrets 用 GitHub Secrets 管理
4. **回滚机制**: 保留最近 10 个版本构建产物，支持一键回滚

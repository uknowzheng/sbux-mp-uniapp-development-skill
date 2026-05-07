# Build & Deploy Full Guide

## Current Build System

### package.json Scripts Structure

```bash
# Multi-platform mini-program builds
app:dev:mp-weixin      # WeChat mini-program development
app:stg:mp-weixin      # WeChat mini-program staging
app:prod:mp-weixin     # WeChat mini-program production
app:dev:mp-alipay      # Alipay mini-program
app:dev:mp-toutiao     # Douyin mini-program

# H5 multi-business-line builds
dev:h5-module-config   # H5 module config development
stg:h5-module-config   # H5 module config staging
dev:h5-generalActivity # H5 general activity
dev:h5-loyalty         # H5 loyalty

# Electronic coupon/ticket
dev:ele-coupon
stg:ele-coupon

# Customization business (offline package)
dev:h5-customization-mod
dev:h5-customization-mop
stg:customization-mod
build:customization-mod
```

### build-script Core Flow

`build-script/platform-builder.js` core logic:

1. Read environment variables
   - `NODE_CUSTOM_ENV`: dev/stg/prod
   - `TARGET_PLATFORM`: mp-weixin/mp-alipay/h5-xxx
   - `BUILD_TYPE`: OfflinePackage (optional)
   - `CURR_MACHINE`: android/ios (offline package)
   - `BUSINESS_TYPE`: MOD/MOP (offline package)

2. Build process
   - Execute platform-specific build scripts
   - Generate pages.json, manifest.json
   - Copy share modules
   - Execute Vue CLI build

3. Output to `dist/` directory

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `NODE_CUSTOM_ENV` | Environment identifier | dev / stg / prod |
| `UNI_PLATFORM` | Platform identifier | mp-weixin / mp-alipay / h5-xxx |
| `BUILD_TYPE` | Build type | OfflinePackage (optional) |
| `CURR_MACHINE` | Device type | android / ios (offline package) |
| `BUSINESS_TYPE` | Business type | MOD / MOP (offline package) |

## CI/CD Pipeline Design

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy-pipeline.yml
name: UniApp Deploy Pipeline

on:
  push:
    branches: [develop, release/*, main]
  pull_request:
    branches: [develop, main]

jobs:
  # Stage 1: Code quality checks
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

  # Stage 2: Build and preview (on PR)
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

  # Stage 3: Deploy to staging
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

  # Stage 4: Deploy to production (requires manual approval)
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

### Deployment Script Example

```bash
#!/bin/bash
# scripts/release-miniprogram.sh

echo "Starting mini-program release..."

# WeChat mini-program
npx miniprogram-ci upload \
  --appid $WECHAT_APPID \
  --private-key $WECHAT_PRIVATE_KEY \
  --project-path dist/build/mp-weixin \
  --version $(node -p "require('./package.json').version") \
  --desc "Production Release"

# Alipay mini-program
npx minidev upload \
  --app-id $ALIPAY_APPID \
  --private-key $ALIPAY_PRIVATE_KEY \
  --project dist/build/mp-alipay \
  --version $(node -p "require('./package.json').version")

echo "Release complete!"
```

## Best Practices

1. **Branch Strategy**: main(production) → develop(staging) → feature/*(feature) → hotfix/*(hotfix)
2. **Automated Checks**: All PRs must pass CI, ESLint zero errors
3. **Security Policy**: Production deployments require manual approval, Secrets managed via GitHub Secrets
4. **Rollback Mechanism**: Keep last 10 version build artifacts, support one-click rollback
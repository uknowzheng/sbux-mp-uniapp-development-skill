# /deploy - Build & Deploy

## Development Workflow

### Step 1: Select Build Command

```bash
# WeChat Mini Program
pnpm run app:dev:mp-weixin       # Development
pnpm run app:stg:mp-weixin       # Staging
pnpm run app:prod:mp-weixin      # Production

# Alipay Mini Program
pnpm run app:dev:mp-alipay

# H5 Business Lines
pnpm run dev:h5-module-config    # Development
pnpm run stg:h5-module-config    # Staging

# Customization (offline package)
pnpm run dev:h5-customization-mod
pnpm run build:customization-mod # Production
```

### Step 2: Set Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `NODE_CUSTOM_ENV` | Environment | dev / stg / prod |
| `UNI_PLATFORM` | Platform | mp-weixin / mp-alipay / h5-xxx |
| `BUILD_TYPE` | Build type | OfflinePackage (optional) |
| `BUSINESS_TYPE` | Business type | MOD / MOP (offline package) |

### Step 3: Memory Optimization (During Development)

```bash
export NODE_OPTIONS="--max-old-space-size=4096"
```

### Step 4: Build Output

Build artifacts are output to the `dist/` directory.

## Build Process Overview

`build-script/platform-builder.js` core flow:
1. Read environment variables → 2. Execute platform build scripts → 3. Generate pages.json/manifest.json → 4. Copy share modules → 5. Vue CLI build → 6. Output to dist/

## CI/CD Pipeline

- **develop branch** → Auto-deploy to staging environment
- **main branch** → Manual approval then deploy to production
- **PR** → Auto lint + test + build verification

## Branch Strategy

`main`(production) → `develop`(staging) → `feature/*`(feature) → `hotfix/*`(hotfix)

## Detailed Reference

→ Read [reference/deployment.md](../reference/deployment.md) for full CI/CD config, GitHub Actions YAML, deployment scripts

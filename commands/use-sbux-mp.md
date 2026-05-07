# /use-sbux-mp - End-to-End Business Feature Development

Develop a complete page feature module based on business requirements, following all project standards.

## Development Workflow

### Step 1: Analyze Requirements, Decompose Modules

After receiving business requirements, decompose into the following modules and determine if needed:

| Module | Needed When | Decision Criteria | Read File |
|--------|------------|-------------------|-----------|
| Service | New API → needed | Read [reference/services.md](../reference/services.md) |
| Store | Cross-page state → needed | Read [reference/store.md](../reference/store.md) |
| Transaction Store | Cart/store/transaction → needed | Read [reference/transaction-store.md](../reference/transaction-store.md) |
| SharedCache | Cross-page temporary data passing → needed | Read [reference/shared-cache.md](../reference/shared-cache.md) |
| Composable | Has business logic → required | Read [reference/composables.md](../reference/composables.md) |
| Component | Has reusable UI blocks → needed | Read [reference/component.md](../reference/component.md) |
| Dialog | Has dialog interactions → needed | Read [reference/dialog.md](../reference/dialog.md) |
| Track | Has tracking requirements → needed | SKILL.md Section 9 |
| Monitor | Key business flows → needed | Read [reference/monitor.md](../reference/monitor.md) |
| Static Assets | Has new image assets → needed | Read [reference/static-assets.md](../reference/static-assets.md) |

**Output**: List the files to be created/modified and their dependency relationships.

### Step 2: Create Bottom-Up, in Dependency Order

Strictly create files in the following order (upper layers depend on lower layers):

```
Service → Store/SharedCache → Composable → Component → Page → Route Registration
```

#### 2.1 Service Layer (If New API)

1. Create `src/services/[domain]/api.js` — define API paths
2. Create `src/services/[domain]/index.js` — functional style exports
3. Multi-channel → create `xxx.weixin.js` / `xxx.h5.js` + routing entry
4. Register in `src/services/index.js`

#### 2.2 Store / SharedCache Layer (If Cross-Page State)

**Store** (reactive state):
1. Create `common/stores/use[Xxx]Store.js` — defineStore pattern
2. Transaction-related → use `useCartStore` / `useShopStore` / `useTransactionStore`

**SharedCache** (lightweight temporary cache):
1. Create `common/sharedCache/modules/[name].js` — defineCache pattern

#### 2.3 Composable Layer (Required)

1. Create sub-composables `composables/use[Feature]Data.js` / `use[Feature]Logic.js` / `use[Feature]Track.js`
2. Create main composable `use[PageName].js` — manage lifecycle + coordinate sub-composables
3. **Constraint**: Sub-composables must not contain lifecycles, cross-composable watches only in main composable

#### 2.4 Component Layer (If Reusable UI)

1. Create `components/[componentName]/index.vue`
2. Complex component → add `use[ComponentName].js`
3. Dialog → use `useAlertDialog` / `useSlideDialog` / `usePromptDialog`
4. Images → use `imgUrl()`

#### 2.5 Page Layer (Required)

1. Create page directory `pages/[pageName]/index.vue` + `use[PageName].js` + `composables/`
2. Legacy page → create same-name directory at same level, keep `.vue` unchanged
3. Vue pattern: no `#ifdef` → `<script setup>`; with `#ifdef` → Options `setup()`
4. Declare empty lifecycles: `onReachBottom` / `onShareAppMessage` / `onPullDownRefresh` / `onPageScroll`

#### 2.6 Track Layer (If Tracking)

1. Create `[package]/track/[pageName].js` — encapsulate tracking functions
2. Naming: `track[Element]Click` / `track[Element]Expose` / `trackPageView`
3. Platform differences `#ifdef` handled inside track functions
4. Call track functions in composables

#### 2.7 Monitor Layer (Key Business Flows)

1. Import `useMonitorReport` from `tingyun/index.js`
2. Report at key points: `monitorReport.info()` (start) / `monitorReport.error()` (failure)
3. Reporting must not block business, throttle high-frequency operations

### Step 3: Register Routes

1. Add page path in `pages.config.js`
2. Add route constant in `common/router.config.js` (grouped by package)
3. Use `navigateTo({ url: XXX.pageName })` for navigation, no hardcoding

### Step 4: Self-Check Checklist

Verify each of the following standards is fully satisfied:

| # | Check Item | Spec Source |
|---|-----------|-------------|
| 1 | All file and directory naming is camelCase | SKILL.md Section 1 |
| 2 | Hook naming use + PascalCase, named exports | SKILL.md Section 1 |
| 3 | Components use directory form, main component index.vue | SKILL.md Section 2 |
| 4 | Sub-composables have no lifecycles | SKILL.md Section 4 |
| 5 | Cross-composable watches only in main composable | SKILL.md Section 4 |
| 6 | Store uses defineStore, direct .value assignment prohibited | SKILL.md Section 8 |
| 7 | Service only does requests + simple transforms, no routing/complex logic | SKILL.md Section 7 |
| 8 | Routes use constants, no hardcoded paths | SKILL.md Section 6 |
| 9 | Images use imgUrl() | SKILL.md Section 8.8 |
| 10 | Tracking encapsulated as functions, split by page | SKILL.md Section 9 |
| 11 | Lifecycles requiring manual declaration have empty methods in .vue | SKILL.md Section 3 |
| 12 | Platform differences use #ifdef, not scattered in business logic | SKILL.md Section 7/9 |

## Output Format

After completing development, output the following:

```markdown
## Feature Module: [Business Feature Name]

### File List
| File Path | Type | Description |
|-----------|------|-------------|
| src/services/xxx/api.js | Service | API definitions |
| src/services/xxx/index.js | Service | Service methods |
| ... | ... | ... |

### Dependency Relationships
Service ← Store/Cache ← Composable ← Component ← Page

### Route Registration
- pages.config.js: added path
- router.config.js: added constant

### Self-Check Results
- [x] All 12 check items passed
```

## Detailed Reference

Read the following reference files on demand for complete standards:

→ [reference/services.md](../reference/services.md) — Service layer full guide + multi-channel routing
→ [reference/store.md](../reference/store.md) — Store full usage
→ [reference/transaction-store.md](../reference/transaction-store.md) — Transaction Store
→ [reference/shared-cache.md](../reference/shared-cache.md) — Cross-page shared cache
→ [reference/composables.md](../reference/composables.md) — Main/sub composable full architecture
→ [reference/component.md](../reference/component.md) — Component full patterns
→ [reference/dialog.md](../reference/dialog.md) — Dialog Composables
→ [reference/static-assets.md](../reference/static-assets.md) — Static assets imgUrl
→ [reference/monitor.md](../reference/monitor.md) — Monitoring full API
→ [reference/deployment.md](../reference/deployment.md) — Build & deploy

# /page - Create New Page

## Development Workflow

### Step 1: Create Page Directory

```bash
pages/
└── [pageName]/              # Page directory (camelCase)
    ├── index.vue            # Page component
    ├── use[PageName].js     # Main composable
    └── composables/         # Sub-composables
```

### Step 2: Write Main Composable

The main composable manages all lifecycles and imports sub-composables (pure logic, no lifecycles). Cross-composable watches are managed only in the main composable.

Named export: `export function useXxx() {}`

### Step 3: Write Page Vue

- **No platform differences** → `<script setup>` + `import { useXxx } from './useXxx'`
- **With `#ifdef` conditional compilation** → Options `setup()` + declare empty lifecycle methods

Lifecycles requiring manual declaration: `onReachBottom` / `onShareAppMessage` / `onPullDownRefresh` / `onPageScroll`

### Step 4: Register Page Route

Add the page path in `pages.config.js`.

## Key Constraints

1. Sub-composables must not contain lifecycles
2. Cross-composable watches are managed only in the main composable
3. Lifecycles requiring manual declaration must have empty methods in .vue
4. Named export: `export function useXxx() {}`

## Detailed Reference

→ Read [reference/composables.md](../reference/composables.md) for full main/sub composable architecture and code examples

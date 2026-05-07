# /component - Create New Component

## Development Workflow

### Step 1: Determine Component Type

| Type | Directory Structure | Use Case |
|------|-------------------|----------|
| Simple component | `cardItem/index.vue` | Display/interaction only |
| Complex component | `orderDetailPanel/index.vue` + `useOrderDetailPanel.js` | Has state logic |
| Component group | `svcPay/payMent.vue` + `payPop.vue` + ... | Multiple related sub-components |

### Step 2: Create Component Directory

```bash
components/
└── [componentName]/         # camelCase
    ├── index.vue            # Main component
    └── use[ComponentName].js # Only needed for complex components
```

### Step 3: Write Component

- **Simple component**: Pass data via props + `$emit` for events, scoped styles
- **Complex component**: Extract hook (`use[ComponentName].js`), `return useXxx(props, emit)` in setup
- **Dialog component**: Support open/close methods, call via ref (see [reference/dialog.md](../reference/dialog.md))

### Step 4: Use Component

Import and use in the page. Pass data via props, events via $emit. Use `imgUrl` for images.

## Key Constraints

1. All components use directory form, main component named `index.vue`
2. Pass data via props, avoid over-reliance on global state
3. Use $emit for events, maintain unidirectional data flow
4. Use scoped styles to avoid style pollution
5. Image fallback: use `imgUrl` function (see [reference/static-assets.md](../reference/static-assets.md))
6. Dialog components: support open/close methods, call via ref

## Detailed Reference

→ Read [reference/component.md](../reference/component.md) for complex component full patterns and examples
→ Read [reference/dialog.md](../reference/dialog.md) for Dialog Composables usage guide

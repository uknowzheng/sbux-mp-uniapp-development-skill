# /composable - Write Composable

## Development Workflow

### Step 1: Determine Layer

| Layer | Location | Responsibility | Lifecycle |
|-------|----------|---------------|-----------|
| Main composable | `pages/[pageName]/use[PageName].js` | Manage lifecycle + coordinate sub-composables | Yes |
| Sub - Data layer | `composables/use[Feature]Data.js` | Data fetching and state management | No |
| Sub - Business layer | `composables/use[Feature]Logic.js` | Business logic processing | No |
| Sub - Utility layer | `composables/use[Feature].js` | Utility function encapsulation | No |
| Sub - Track layer | `composables/use[Feature]Track.js` | Tracking logic | No |
| Package shared | `[package]/composables/` | Multi-page shared within package | No |
| Global shared | `common/composables/` | Cross-package common | No |

### Step 2: Write Sub-Composable

Pure logic, no `@dcloudio/uni-app` lifecycles. Internal watches only monitor own state. Named export: `export function useXxx() {}`

### Step 3: Write Main Composable

Manage lifecycles (onLoad/onShow/onUnload) + import sub-composables + cross-composable watches. Sub-composables can depend on other sub-composables (dependency injection).

### Step 4: Use in Page

Page Vue file imports the main composable, exposes to template via `return useXxx()`.

## Key Constraints

1. Sub-composables must not contain lifecycles
2. Cross-composable watches are managed only in the main composable
3. Internal watches in sub-composables only monitor their own state
4. Named export: `export function useXxx() {}`, default export prohibited
5. All reactive data uses ref/computed, primitive `let` variables prohibited

## Common Mistakes

- Sub-composable contains `onLoad` → prohibited
- `export default function useXxx()` → prohibited, use named export
- `let count = 0` → should use `ref(0)`

## Detailed Reference

→ Read [reference/composables.md](../reference/composables.md) for full architecture, watch management conventions, correct/incorrect examples

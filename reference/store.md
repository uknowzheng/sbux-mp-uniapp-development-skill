# Store Full Usage Guide

## Directory Structure

```bash
common/stores/
├── defineStore.js            # defineStore factory function
├── useCommonStore.js         # Common state
├── useMainStore.js           # Home state
├── useEcStore.js             # EC business
└── ...
```

## Transition Phase Notes

Currently all Store data access still comes from Vuex, only syntax is extended. defineStore is a wrapper around Vuex.

## Store Definition Example

```javascript
// common/stores/useMainStore.js
import { defineStore } from './defineStore.js';
import { computed } from '@vue/composition-api';

// Mutation constants (internal use only, not exported)
const UPDATE_MAIN_RISKINFO = 'UPDATE_MAIN_RISKINFO';
const RED_SPOT_INFO = 'RED_SPOT_INFO';

export const useMainStore = defineStore('main', ({ state, commit }) => {
  // Getters - wrapped with computed
  const riskInfo = computed(() => state.riskInfo.value);
  const redSpotInfo = computed(() => state.redSpotInfo.value);

  // Actions - wrapping commit
  const setRiskInfo = (data) => commit(UPDATE_MAIN_RISKINFO, data);
  const setRedSpotInfo = (data) => commit(RED_SPOT_INFO, data);

  return {
    riskInfo,
    redSpotInfo,
    setRiskInfo,
    setRedSpotInfo,
  };
});
```

## Using in Page

```vue
<template>
  <view>
    <text>Risk info: {{ riskInfo }}</text>
    <text>Red spot: {{ redSpotInfo }}</text>
    <button @click="handleClick">Update risk</button>
  </view>
</template>

<script>
import { useMainStore } from 'common/stores/useMainStore';

export default {
  setup() {
    // 1. Get store instance
    const mainStore = useMainStore();

    // 2. Read state (JS requires .value)
    console.log(mainStore.riskInfo.value);
    console.log(mainStore.redSpotInfo.value);

    // 3. Modify state
    const handleClick = () => {
      mainStore.setRiskInfo({ data: { level: 'high' } });
    };

    // 4. Return to template (template auto-unwraps, no .value needed)
    return {
      riskInfo: mainStore.riskInfo,
      redSpotInfo: mainStore.redSpotInfo,
      handleClick,
    };
  },
};
</script>
```

## Using in Composable/Hook

```javascript
// composables/useRiskCheck.js
import { watch } from '@vue/composition-api';
import { useMainStore } from 'common/stores/useMainStore';

export function useRiskCheck() {
  const mainStore = useMainStore();

  // Read state
  const riskInfo = mainStore.riskInfo;

  // Watch changes
  watch(
    () => riskInfo.value,
    (newVal) => {
      console.log('Risk info changed:', newVal);
    }
  );

  // Business method
  const fetchRiskInfo = async () => {
    const res = await riskService.check();
    mainStore.setRiskInfo({ data: res });
  };

  return { riskInfo, fetchRiskInfo };
}
```

## Usage Quick Reference

| Scenario | Pattern |
|----------|---------|
| Get store | `const mainStore = useMainStore()` |
| Read value in JS | `mainStore.riskInfo.value` |
| Read value in template | `{{ riskInfo }}` (auto-unwrapped) |
| Modify state | `mainStore.setRiskInfo({ data: xxx })` |
| Watch changes | `watch(() => mainStore.riskInfo.value, ...)` |

## Key Points

1. **Getter needs `.value`**: Must use `.value` when reading values in JS logic
2. **Template auto-unwraps**: Use directly in `<template>`, Vue auto-unwraps
3. **Modify via exported methods**: Parameter format is consistent with original Vuex mutations
4. **Direct assignment prohibited**: `mainStore.riskInfo.value = xxx` is strictly prohibited, must use exported methods

## Correct vs Incorrect

```javascript
// Wrong: Direct read (gets ComputedRef object)
console.log(mainStore.riskInfo);

// Correct: Use .value
console.log(mainStore.riskInfo.value);

// Wrong: Direct assignment
mainStore.riskInfo.value = { level: 'low' };

// Correct: Use exported method
mainStore.setRiskInfo({ data: { level: 'low' } });
```

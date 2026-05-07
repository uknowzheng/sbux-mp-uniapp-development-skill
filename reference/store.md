# Store 完整用法指南

## 目录结构

```bash
common/stores/
├── defineStore.js            # defineStore 工厂函数
├── useCommonStore.js         # 通用状态
├── useMainStore.js           # 首页状态
├── useEcStore.js             # EC 业务
└── ...
```

## 过渡阶段说明

当前所有 Store 数据访问能力仍来自 Vuex，仅语法扩展。defineStore 是对 Vuex 的封装。

## Store 定义示例

```javascript
// common/stores/useMainStore.js
import { defineStore } from './defineStore.js';
import { computed } from '@vue/composition-api';

// Mutation 常量（仅内部使用，不导出）
const UPDATE_MAIN_RISKINFO = 'UPDATE_MAIN_RISKINFO';
const RED_SPOT_INFO = 'RED_SPOT_INFO';

export const useMainStore = defineStore('main', ({ state, commit }) => {
  // Getters - 使用 computed 包装
  const riskInfo = computed(() => state.riskInfo.value);
  const redSpotInfo = computed(() => state.redSpotInfo.value);

  // Actions - 包装 commit
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

## 在页面中使用

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
    // 1. 获取 store 实例
    const mainStore = useMainStore();

    // 2. 读取状态（JS 中需要 .value）
    console.log(mainStore.riskInfo.value);
    console.log(mainStore.redSpotInfo.value);

    // 3. 修改状态
    const handleClick = () => {
      mainStore.setRiskInfo({ data: { level: 'high' } });
    };

    // 4. 返回给模板（模板自动解包，无需 .value）
    return {
      riskInfo: mainStore.riskInfo,
      redSpotInfo: mainStore.redSpotInfo,
      handleClick,
    };
  },
};
</script>
```

## 在 Composable/Hook 中使用

```javascript
// composables/useRiskCheck.js
import { watch } from '@vue/composition-api';
import { useMainStore } from 'common/stores/useMainStore';

export function useRiskCheck() {
  const mainStore = useMainStore();

  // 读取状态
  const riskInfo = mainStore.riskInfo;

  // 监听变化
  watch(
    () => riskInfo.value,
    (newVal) => {
      console.log('Risk info changed:', newVal);
    }
  );

  // 业务方法
  const fetchRiskInfo = async () => {
    const res = await riskService.check();
    mainStore.setRiskInfo({ data: res });
  };

  return { riskInfo, fetchRiskInfo };
}
```

## 用法速查

| 场景 | 写法 |
|------|------|
| 获取 store | `const mainStore = useMainStore()` |
| JS 中读取值 | `mainStore.riskInfo.value` |
| 模板中读取 | `{{ riskInfo }}`（自动解包） |
| 修改状态 | `mainStore.setRiskInfo({ data: xxx })` |
| Watch 变化 | `watch(() => mainStore.riskInfo.value, ...)` |

## 关键要点

1. **Getter 需要 `.value`**：JS 逻辑中读取值时，必须使用 `.value`
2. **模板自动解包**：在 `<template>` 中直接使用，Vue 自动解包
3. **导出方法修改**：参数格式与原 Vuex mutation 一致
4. **禁止直接赋值**：`mainStore.riskInfo.value = xxx` 严禁，必须用导出方法

## 正确 vs 错误

```javascript
// 错误：直接读取（得到 ComputedRef 对象）
console.log(mainStore.riskInfo);

// 正确：使用 .value
console.log(mainStore.riskInfo.value);

// 错误：直接赋值
mainStore.riskInfo.value = { level: 'low' };

// 正确：使用导出方法
mainStore.setRiskInfo({ data: { level: 'low' } });
```

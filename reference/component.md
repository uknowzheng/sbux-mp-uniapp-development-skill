# 组件完整开发指南

## 复杂页面组件拆分模式

对于复杂页面（如支付页），组件拆分为以下几类：

### 1. 业务逻辑组件

```bash
components/
├── paymentList/           # 支付方式列表
│   └── index.vue
├── orderType/             # 订单类型选择（即刻取单/预约取单）
│   └── index.vue
├── address/               # 地址选择（门店/配送地址）
│   └── index.vue
└── submit/                # 提交按钮区域
    └── index.vue
```

**示例：paymentList 组件**

```vue
<!-- components/paymentList/index.vue -->
<template>
  <view class="payment-list">
    <view
      v-for="method in paymentMethods"
      :key="method.paymentMethodCode"
      class="payment-item"
      :class="{ active: selectedCode === method.paymentMethodCode }"
      @click="handleSelect(method)"
    >
      <image class="payment-icon" :src="imgUrl(method.iconUrl)" mode="aspectFit" />
      <view class="payment-info">
        <text class="payment-name">{{ method.paymentMethodName }}</text>
        <text v-if="method.balance" class="payment-balance">
          余额: ¥{{ formatAmount(method.balance) }}
        </text>
      </view>
      <view class="check-icon" v-if="selectedCode === method.paymentMethodCode">
        <text class="icon-check">✓</text>
      </view>
    </view>
  </view>
</template>

<script>
import { imgUrl } from 'common/utils/imgUrl.js';

export default {
  props: {
    paymentMethods: { type: Array, default: () => [] },
    selectedCode: { type: String, default: '' },
  },
  emits: ['select'],
  setup(props, { emit }) {
    const handleSelect = (method) => { emit('select', method); };
    const formatAmount = (amount) => (amount / 100).toFixed(2);
    return { imgUrl, handleSelect, formatAmount };
  },
};
</script>

<style lang="less" scoped>
.payment-list {
  .payment-item {
    display: flex;
    align-items: center;
    padding: 20rpx 30rpx;
    background: #fff;
    border-bottom: 1rpx solid #eee;
    &.active { background: #f5f5f5; }
    .payment-icon { width: 60rpx; height: 60rpx; margin-right: 20rpx; }
    .payment-info {
      flex: 1;
      .payment-name { font-size: 28rpx; color: #333; }
      .payment-balance { font-size: 24rpx; color: #999; margin-left: 10rpx; }
    }
    .check-icon {
      width: 40rpx; height: 40rpx; background: #007aff; border-radius: 50%;
      display: flex; align-items: center; justify-content: center;
      .icon-check { color: #fff; font-size: 24rpx; }
    }
  }
}
</style>
```

### 2. 交互组件（Popups/Modals）

```bash
components/
├── SVCCardListPopup/      # 星礼卡列表弹窗
│   └── index.vue
├── PayMethodPopup/        # 支付方式弹窗
│   └── index.vue
├── CompleteAddressDialog/ # 地址补全弹窗
│   └── index.vue
└── OtherStoreConfimPopup/ # 跨门店确认弹窗
    └── index.vue
```

**弹窗组件设计模式**：支持 open/close 方法，通过 ref 调用

```vue
<!-- components/SVCCardListPopup/index.vue -->
<template>
  <uni-popup ref="popup" type="bottom" @change="onPopupChange">
    <view class="svc-card-popup">
      <view class="popup-header">
        <text class="title">选择星礼卡</text>
        <text class="close-btn" @click="close">×</text>
      </view>
      <scroll-view class="card-list" scroll-y>
        <view v-for="card in svcCards" :key="card.cardNumber" class="card-item"
          :class="{ active: selectedCardNo === card.cardNumber }" @click="selectCard(card)">
          <view class="card-info">
            <text class="card-name">{{ card.cardName }}</text>
            <text class="card-no">**** {{ card.cardNumber.slice(-4) }}</text>
          </view>
        </view>
      </scroll-view>
    </view>
  </uni-popup>
</template>

<script>
import { ref } from '@vue/composition-api';

export default {
  props: {
    svcCards: { type: Array, default: () => [] },
    selectedCardNo: { type: String, default: '' },
  },
  emits: ['select', 'confirm', 'add-card'],
  setup(props, { emit }) {
    const popup = ref(null);
    const tempSelectedCard = ref(null);

    const open = () => { popup.value?.open(); tempSelectedCard.value = null; };
    const close = () => { popup.value?.close(); };
    const selectCard = (card) => { tempSelectedCard.value = card; emit('select', card); };
    const confirm = () => { emit('confirm', tempSelectedCard.value); close(); };

    return { popup, open, close, selectCard, confirm };
  },
};
</script>
```

### 3. 容器组件

布局和样式容器，如 `addressAndOrderContain`、`scrollFloatButton`

### 4. 选项行组件

```bash
components/
└── optionRow/
    ├── index.vue
    ├── priceRow.vue          # 价格行
    ├── radioRow.vue          # 单选行
    └── selectRow.vue         # 选择行
```

## 复杂组件带 Hook 模式

### Hook 文件

```javascript
// components/orderDetailPanel/useOrderDetailPanel.js
import { ref, computed, onMounted } from '@vue/composition-api';

export function useOrderDetailPanel(props, emit) {
  const orderData = ref(null);
  const loading = ref(false);
  const isExpanded = ref(false);

  const canCancel = computed(() => {
    return orderData.value?.status === 1 && isLogin.value;
  });

  const priceText = computed(() => {
    if (!orderData.value) return '';
    return `¥${(orderData.value.totalPrice / 100).toFixed(2)}`;
  });

  const fetchOrder = async () => {
    if (!props.orderId) return;
    loading.value = true;
    try {
      const res = await orderService.getDetail({ orderId: props.orderId });
      orderData.value = res;
      emit('loaded', res);
    } catch (e) {
      uni.showToast({ title: e.message, icon: 'none' });
      emit('error', e);
    } finally {
      loading.value = false;
    }
  };

  onMounted(() => { fetchOrder(); });

  return { orderData, loading, isExpanded, canCancel, priceText, fetchOrder };
}
```

### 组件文件

```vue
<!-- components/orderDetailPanel/index.vue -->
<template>
  <view class="order-detail-panel">
    <view v-if="loading" class="loading">Loading...</view>
    <template v-else-if="orderData">
      <view class="order-header" @click="isExpanded = !isExpanded">
        <text class="order-name">{{ orderData.name }}</text>
        <text class="order-price">{{ priceText }}</text>
      </view>
      <view v-if="isExpanded" class="order-detail">
        <view v-for="item in orderData.items" :key="item.id" class="order-item">
          {{ item.name }} x {{ item.quantity }}
        </view>
      </view>
      <slot name="actions" :order="orderData" :canCancel="canCancel">
        <button v-if="canCancel" class="cancel-btn" @click="handleCancel">Cancel Order</button>
      </slot>
    </template>
  </view>
</template>

<script>
import { useOrderDetailPanel } from './useOrderDetailPanel';

export default {
  props: {
    orderId: { type: String, required: true },
    mode: { type: String, default: 'normal' },
  },
  emits: ['loaded', 'error', 'cancelled'],
  setup(props, { emit }) {
    return useOrderDetailPanel(props, emit);
  },
};
</script>
```

## 最佳实践总结

1. **目录形式**：所有组件使用目录形式，主组件命名 `index.vue`
2. **Hook 命名**：`use[ComponentName].js` 用于复杂组件
3. **Props 验证**：使用正确的 prop 类型和验证
4. **事件发射**：使用 emit 进行父组件通信
5. **Slot 使用**：使用 slots 提供灵活内容
6. **Scoped 样式**：始终使用 scoped 样式
7. **性能**：避免不必要的 re-render，使用 computed 属性
8. **复杂组件拆分**：按功能模块拆分（paymentList、orderType、address 等）
9. **弹窗组件**：支持 open/close 方法，通过 ref 调用
10. **图片兜底**：使用 imgUrl 函数，在图片加载失败时提供兜底图

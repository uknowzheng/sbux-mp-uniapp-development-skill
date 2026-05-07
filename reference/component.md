# Component Full Development Guide

## Complex Page Component Splitting Patterns

For complex pages (e.g., payment page), components are split into the following categories:

### 1. Business Logic Components

```bash
components/
├── paymentList/           # Payment method list
│   └── index.vue
├── orderType/             # Order type selection (immediate/scheduled pickup)
│   └── index.vue
├── address/               # Address selection (store/delivery address)
│   └── index.vue
└── submit/                # Submit button area
    └── index.vue
```

**Example: paymentList Component**

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
          Balance: ¥{{ formatAmount(method.balance) }}
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

### 2. Interactive Components (Popups/Modals)

```bash
components/
├── SVCCardListPopup/      # SVC card list popup
│   └── index.vue
├── PayMethodPopup/        # Payment method popup
│   └── index.vue
├── CompleteAddressDialog/ # Address completion dialog
│   └── index.vue
└── OtherStoreConfimPopup/ # Cross-store confirmation popup
    └── index.vue
```

**Dialog Component Design Pattern**: Support open/close methods, call via ref

```vue
<!-- components/SVCCardListPopup/index.vue -->
<template>
  <uni-popup ref="popup" type="bottom" @change="onPopupChange">
    <view class="svc-card-popup">
      <view class="popup-header">
        <text class="title">Select SVC Card</text>
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

### 3. Container Components

Layout and style containers, e.g., `addressAndOrderContain`, `scrollFloatButton`

### 4. Option Row Components

```bash
components/
└── optionRow/
    ├── index.vue
    ├── priceRow.vue          # Price row
    ├── radioRow.vue          # Radio row
    └── selectRow.vue         # Select row
```

## Complex Component with Hook Pattern

### Hook File

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

### Component File

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

## Best Practices Summary

1. **Directory form**: All components use directory form, main component named `index.vue`
2. **Hook naming**: `use[ComponentName].js` for complex components
3. **Props validation**: Use correct prop types and validation
4. **Event emission**: Use emit for parent component communication
5. **Slot usage**: Use slots for flexible content
6. **Scoped styles**: Always use scoped styles
7. **Performance**: Avoid unnecessary re-renders, use computed properties
8. **Complex component splitting**: Split by functional module (paymentList, orderType, address, etc.)
9. **Dialog components**: Support open/close methods, call via ref
10. **Image fallback**: Use imgUrl function for image loading failure fallback

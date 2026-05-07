# Complete Code Examples

## Example 1: Payment Page Full Chain

### Page Entry index.vue

```vue
<template>
  <view class="pay-page">
    <!-- #ifdef H5 -->
    <h5-navigation />
    <!-- #endif -->
    <payment-list
      :payment-methods="payMethodList"
      :selected-code="selectedPayMethod"
      @select="handlePayMethodSelect"
    />
    <submit-area :can-submit="canSubmit" @submit="handleSubmit" />
  </view>
</template>

<script>
import { usePay } from './usePay';
// #ifdef H5
import H5Navigation from '../components/h5Header/index.vue';
// #endif

export default {
  components: {
    // #ifdef H5
    H5Navigation,
    // #endif
  },
  // Lifecycles requiring manual declaration (empty methods, actual logic in hook)
  onReachBottom() {},
  onShareAppMessage() {},
  onPullDownRefresh() {},
  onPageScroll() {},
  setup() {
    return usePay();
  }
};
</script>
```

### Main Composable usePay.js

```javascript
import { onLoad, onShow, onUnload, onPageScroll } from '@dcloudio/uni-app';
import { watch, computed } from '@vue/composition-api';
import { useOrder } from 'shares/transaction/composables/useOrder.js';
import { usePayMethod } from './composables/usePayMethod.js';
import { usePayTrack } from './composables/usePayTrack.js';

export function usePay() {
  // 1. Lifecycle management
  onLoad((params) => { fetchReviewData(params); });
  onShow(() => { trackPageView(); });
  onUnload(() => { cleanup(); });

  // 2. Import sub-composables
  const { orderReviewData, fetchReviewData, isCartEmpty } = useOrder();
  const { selectedPayMethod, payMethodList, fetchPayMethod, submitOrder } = usePayMethod();
  const { trackPageView, trackPaymentToolAction } = usePayTrack();

  // 3. Cross-composable watch
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal);
    }
  });

  // 4. Composed computed property
  const canSubmit = computed(() => {
    return orderReviewData.value && selectedPayMethod.value && !isCartEmpty.value;
  });

  // 5. Composed methods
  const handleSubmit = async () => {
    if (!canSubmit.value) return;
    await submitOrder();
  };

  const handlePayMethodSelect = (method) => {
    selectedPayMethod.value = method.paymentMethodCode;
  };

  return {
    orderReviewData, selectedPayMethod, payMethodList,
    canSubmit, handleSubmit, handlePayMethodSelect,
  };
}
```

### Sub-Composable useOrder.js

```javascript
import { ref, computed } from '@vue/composition-api';
import { orderReview } from 'shares/transaction/services/order/index.js';

export function useOrder() {
  const orderReviewData = ref(null);

  const isCartEmpty = computed(() => !orderReviewData.value?.productLines?.length);
  const payPrice = computed(() => orderReviewData.value?.payPrice || 0);

  const fetchReviewData = async (params) => {
    const res = await orderReview(params);
    orderReviewData.value = res;
  };

  return { orderReviewData, isCartEmpty, payPrice, fetchReviewData };
}
```

## Example 2: Service + Store + Tracking Full Chain

```javascript
// composables/useCouponList.js
import { ref, computed } from '@vue/composition-api';
import { getCouponList } from 'services/coupon';
import { useMainStore } from 'common/stores/useMainStore';
import { trackCouponClick, trackPageView } from '../track/couponList';

export function useCouponList() {
  const couponList = ref([]);
  const loading = ref(false);
  const mainStore = useMainStore();

  const availableCoupons = computed(() =>
    couponList.value.filter(c => c.status === 'available')
  );

  const fetchCoupons = async () => {
    loading.value = true;
    try {
      const res = await getCouponList({ userId: mainStore.riskInfo.value?.userId });
      couponList.value = res.list;
    } catch (error) {
      uni.showToast({ title: error.message, icon: 'none' });
    } finally {
      loading.value = false;
    }
  };

  const handleCouponClick = (coupon) => {
    trackCouponClick(coupon);
    // Business logic...
  };

  return { couponList, availableCoupons, loading, fetchCoupons, handleCouponClick };
}
```

## Example 3: Store Definition and Usage

```javascript
// common/stores/useOrderStore.js
import { defineStore } from './defineStore.js';
import { computed } from '@vue/composition-api';

const UPDATE_ORDER_LIST = 'UPDATE_ORDER_LIST';
const UPDATE_CURRENT_ORDER = 'UPDATE_CURRENT_ORDER';

export const useOrderStore = defineStore('order', ({ state, commit }) => {
  const orderList = computed(() => state.orderList.value);
  const currentOrder = computed(() => state.currentOrder.value);

  const setOrderList = (data) => commit(UPDATE_ORDER_LIST, data);
  const setCurrentOrder = (data) => commit(UPDATE_CURRENT_ORDER, data);

  return { orderList, currentOrder, setOrderList, setCurrentOrder };
});

// Using in a composable
import { useOrderStore } from 'common/stores/useOrderStore';

export function useOrderData() {
  const orderStore = useOrderStore();

  const fetchOrders = async () => {
    const res = await orderService.getOrderList();
    orderStore.setOrderList({ data: res.list });
  };

  return { orderList: orderStore.orderList, fetchOrders };
}
```

## Example 4: Track Function Definition and Invocation

```javascript
// track/couponList.js
import { newSendTrackEx, newSaTrack, saTrack } from 'common/services/sdk/export.js';

/**
 * Coupon list page view tracking
 */
export function trackPageView() {
  // #ifdef MP-WEIXIN
  newSendTrackEx('COUPON_LIST_VIEW', 'COUPON_LIST_PAGE', {}, true);
  // #endif

  // #ifdef H5
  saTrack('COUPON_LIST_VIEW', { SCREEN_NAME: 'COUPON_LIST_PAGE' });
  // #endif
}

/**
 * Coupon click tracking
 */
export function trackCouponClick(coupon) {
  // #ifdef MP-WEIXIN
  newSendTrackEx('COUPON_CLICK', 'COUPON_LIST_PAGE', {
    COUPON_ID: coupon.couponCode,
    COUPON_NAME: coupon.couponName,
    COUPON_STATUS: coupon.status,
  }, true);
  // #endif
}
```

## Example 5: Monitoring Integration

```javascript
import { useMonitorReport } from 'tingyun/index.js';

export function useOrderSubmit() {
  const monitorReport = useMonitorReport();

  const submitOrder = async (orderInfo) => {
    try {
      monitorReport.info({
        name: 'order:submit:start',
        message: 'Starting order submission',
        orderId: orderInfo.id
      });

      const result = await orderService.submit(orderInfo);

      monitorReport.info({
        name: 'order:submit:success',
        message: 'Order submitted successfully',
        orderId: orderInfo.id
      });

      return result;
    } catch (error) {
      monitorReport.error({
        name: 'order:submit:error',
        message: 'Order submission failed',
        orderId: orderInfo.id,
        error: error.message,
        stack: error.stack
      });
      throw error;
    }
  };

  return { submitOrder };
}
```

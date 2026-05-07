# 完整代码示例

## 示例1：支付页面完整链路

### 页面入口 index.vue

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
  // 需要手动声明的生命周期（空方法，实际逻辑在 hook）
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

### 主 Composable usePay.js

```javascript
import { onLoad, onShow, onUnload, onPageScroll } from '@dcloudio/uni-app';
import { watch, computed } from '@vue/composition-api';
import { useOrder } from 'shares/transaction/composables/useOrder.js';
import { usePayMethod } from './composables/usePayMethod.js';
import { usePayTrack } from './composables/usePayTrack.js';

export function usePay() {
  // 1. 生命周期管理
  onLoad((params) => { fetchReviewData(params); });
  onShow(() => { trackPageView(); });
  onUnload(() => { cleanup(); });

  // 2. 引入子 composables
  const { orderReviewData, fetchReviewData, isCartEmpty } = useOrder();
  const { selectedPayMethod, payMethodList, fetchPayMethod, submitOrder } = usePayMethod();
  const { trackPageView, trackPaymentToolAction } = usePayTrack();

  // 3. 跨 composables 的 watch
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal);
    }
  });

  // 4. 组合计算属性
  const canSubmit = computed(() => {
    return orderReviewData.value && selectedPayMethod.value && !isCartEmpty.value;
  });

  // 5. 组合方法
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

### 子 Composable useOrder.js

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

## 示例2：服务调用 + Store + 埋点完整链路

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
    // 业务逻辑...
  };

  return { couponList, availableCoupons, loading, fetchCoupons, handleCouponClick };
}
```

## 示例3：Store 定义与使用

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

// 在 composable 中使用
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

## 示例4：埋点函数定义与调用

```javascript
// track/couponList.js
import { newSendTrackEx, newSaTrack, saTrack } from 'common/services/sdk/export.js';

/**
 * 优惠券列表页浏览埋点
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
 * 优惠券点击埋点
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

## 示例5：监控上报集成

```javascript
import { useMonitorReport } from 'tingyun/index.js';

export function useOrderSubmit() {
  const monitorReport = useMonitorReport();

  const submitOrder = async (orderInfo) => {
    try {
      monitorReport.info({
        name: 'order:submit:start',
        message: '开始提交订单',
        orderId: orderInfo.id
      });

      const result = await orderService.submit(orderInfo);

      monitorReport.info({
        name: 'order:submit:success',
        message: '订单提交成功',
        orderId: orderInfo.id
      });

      return result;
    } catch (error) {
      monitorReport.error({
        name: 'order:submit:error',
        message: '订单提交失败',
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

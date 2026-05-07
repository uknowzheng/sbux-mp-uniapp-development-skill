# Composable Full Architecture Guide

## Main/Sub Composable Layering Architecture

### Main Composable: Lifecycle Manager

**Responsibility**: Manage all lifecycle hooks, coordinate all sub-composables

**Characteristics**:
- Contains all `onLoad`, `onShow`, `onUnload`, `onPageScroll` and other lifecycles
- Imports all sub-composables
- Manages cross-composable `watch` listeners
- Composes all logic, exposes for page usage

```javascript
// pages/pay/usePay.js - Main composable
import { onLoad, onShow, onUnload, onPageScroll } from '@dcloudio/uni-app';
import { ref, watch, computed } from '@vue/composition-api';

import { useOrder } from 'shares/transaction/composables/useOrder.js';
import { usePayMethod } from './composables/usePayMethod.js';
import { useStoreInfo } from 'shares/transaction/composables/useStoreInfo.js';
import { usePayTrack } from './composables/usePayTrack.js';
import { useScroll } from './composables/useScroll.js';

export const usePay = () => {
  // 1. Lifecycle management (only in main composable)
  onLoad((params) => {
    fetchPayMethod(params);
  });

  onShow(() => {
    trackPageView();
  });

  onPageScroll((e) => {
    handlePageScroll(e);
  });

  onUnload(() => {
    cleanup();
  });

  // 2. Import all sub-composables (pure logic, no lifecycles)
  const { orderReviewData, fetchReviewData } = useOrder();
  const { selectedPayMethod, payMethodList, submitOrder } = usePayMethod();
  const { isNextDayOrder, currentStore } = useStoreInfo();
  const { trackPayResult, trackPaymentToolAction } = usePayTrack();
  const { isShow: isToOrderShow, scrollToTarget: scrollToOrder } = useScroll();

  // 3. Cross-composable watch management (only in main composable)
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal === PaymentType.SVC ? 'SVC Card' : 'WeChat');
    }
  });

  watch(orderReviewData, (newVal) => {
    if (newVal) {
      fetchPayMethod(newVal.channel, newVal.storeId, newVal.payPrice);
    }
  });

  // 4. Composed computed properties (depending on multiple sub-composables)
  const canSubmit = computed(() => {
    return isLogin.value &&
           orderReviewData.value &&
           selectedPayMethod.value &&
           !isDisabledSubmit.value;
  });

  // 5. Composed methods
  const handleSubmit = async () => {
    if (!checkAgreementCheck()) return;
    await submitOrder();
  };

  // 6. Return all needed state and methods
  return {
    orderReviewData, selectedPayMethod, payMethodList,
    isNextDayOrder, currentStore, isToOrderShow,
    canSubmit, handleSubmit, scrollToOrder, fetchReviewData,
  };
};
```

### Sub-Composable: Pure Business Logic

**Responsibility**: Handle single business logic, no lifecycles

**Characteristics**:
- Does not contain `onLoad`, `onShow` and other lifecycles
- Only contains `ref`, `computed`, `methods`
- Can depend on other sub-composables
- Can depend on store

#### Data Layer Sub-Composable

```javascript
// useOrder.js - Pure data logic
import { ref, computed } from '@vue/composition-api';
import { orderReview, orderApply } from 'shares/transaction/services/order/index.js';

export const useOrder = () => {
  const orderReviewData = ref(null);
  const orderApplyData = ref(null);

  const isCartEmpty = computed(() => {
    return !orderReviewData.value?.productLines?.length;
  });

  const payPrice = computed(() => {
    return orderReviewData.value?.payPrice || 0;
  });

  const fetchReviewData = async (params) => {
    const res = await orderReview(params);
    orderReviewData.value = res;
  };

  const fetchApplyData = async (params) => {
    const res = await orderApply(params);
    orderApplyData.value = res;
  };

  return { orderReviewData, orderApplyData, isCartEmpty, payPrice, fetchReviewData, fetchApplyData };
};
```

#### Depending on Other Sub-Composables

```javascript
// usePayMethod.js - Depends on other sub-composables
import { ref, computed } from '@vue/composition-api';
import { getPayMethod, getSVCCardList } from 'shares/transaction/services/payment/index.js';
import { useStoreInfo } from 'shares/transaction/composables/useStoreInfo.js';
import { useOrder } from 'shares/transaction/composables/useOrder.js';

export const usePayMethod = () => {
  const { isNextDayOrder } = useStoreInfo();
  const { currentSVCCardInfo, setSelectedPayMethod } = useOrder();

  const isPayMethodPopupShow = ref(false);
  const payMethodList = ref([]);
  const selectedPayMethod = ref(null);

  const canUseSVCPay = computed(() => {
    return payMethodList.value.some(
      method => method.starPaymentType === 'SVC' && method.enable
    );
  });

  const fetchPayMethod = async (channel, storeId, payPrice) => {
    const res = await getPayMethod({ channel, storeId, payPrice });

    if (isNextDayOrder.value) {
      setSelectedPayMethod('WECHAT');
    } else if (canUseSVCPay.value) {
      setSelectedPayMethod('SVC');
      await fetchSVCCardList(payPrice);
    } else {
      setSelectedPayMethod('WECHAT');
    }

    payMethodList.value = res.paymentMethods;
  };

  const submitOrder = async () => {
    if (selectedPayMethod.value === 'SVC') {
      return await svcPay({ cardNo: currentSVCCardInfo.value.cardNumber });
    } else {
      return await payForWx({ orderId: orderId.value });
    }
  };

  return { isPayMethodPopupShow, payMethodList, selectedPayMethod, canUseSVCPay, fetchPayMethod, submitOrder };
};
```

## Watch Management Conventions

### Cross-Composable Watches (Managed in Main Composable)

```javascript
// usePay.js
export const usePay = () => {
  const { selectedPayMethod } = usePayMethod();
  const { trackPaymentToolAction } = usePayTrack();
  const { orderReviewData } = useOrder();
  const { fetchPayMethod } = usePayMethod();

  // Cross-composable watches only in main composable
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal === PaymentType.SVC ? 'SVC Card' : 'WeChat');
    }
  });

  watch(orderReviewData, (newVal) => {
    if (newVal) {
      fetchPayMethod(newVal.channel, newVal.storeId, newVal.payPrice);
    }
  });

  return { selectedPayMethod, orderReviewData };
};
```

### Internal Watches within a Composable (Managed in Sub-Composable)

```javascript
// useOrder.js - Sub-composable manages internal watch
export const useOrder = () => {
  const orderReviewData = ref(null);

  // Internal watch managed in sub-composable
  watch(orderReviewData, (newVal) => {
    if (newVal) {
      console.log('Order review data updated:', newVal);
    }
  });

  return { orderReviewData };
};
```

## Incorrect Examples

```javascript
// Wrong: Sub-composable contains lifecycle
export function useOrderData() {
  onLoad(() => { fetchData(); }); // Prohibited!
  return { data };
}

// Wrong: Default export
export default function useOrderDetail() {}

// Wrong: Non-reactive
export function useCounter() {
  let count = 0; // Should use ref
  return { count };
}

// Wrong: Directly modifying props
export function useProps(props) {
  props.value = 'new value'; // Don't modify directly
}

// Wrong: Unclear responsibility
export function useEverything() {
  // Data fetching, business logic, tracking, routing all mixed together
}
```

## Correct Examples

```javascript
// Correct: Main composable manages lifecycle
export function usePay() {
  onLoad(() => { /* initialize */ });
  onShow(() => { /* refresh */ });

  const { orderReviewData } = useOrder();
  const { selectedPayMethod } = usePayMethod();

  watch(selectedPayMethod, (newVal) => {
    trackPaymentToolAction(newVal);
  });

  return { orderReviewData, selectedPayMethod };
}

// Correct: Sub-composable pure logic
export function useOrder() {
  const orderReviewData = ref(null);
  const fetchReviewData = async (params) => {
    orderReviewData.value = await orderReview(params);
  };
  return { orderReviewData, fetchReviewData };
}

// Correct: Don't modify props, use local ref
export function useProps(props) {
  const localValue = ref(props.value);
  return { localValue };
}

// Correct: Single responsibility
export function useOrderData() { /* Only data fetching */ }
export function useOrderLogic() { /* Only business logic */ }
```

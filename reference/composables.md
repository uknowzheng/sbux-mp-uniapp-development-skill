# Composable 完整架构指南

## 主/子 Composable 分层架构

### 主 Composable：生命周期管理者

**职责**：管理所有生命周期钩子，协调所有子 composables

**特点**：
- 包含所有 `onLoad`, `onShow`, `onUnload`, `onPageScroll` 等生命周期
- 引入所有子 composables
- 管理跨 composables 的 `watch` 监听
- 组合所有逻辑，暴露给页面使用

```javascript
// pages/pay/usePay.js - 主 composable
import { onLoad, onShow, onUnload, onPageScroll } from '@dcloudio/uni-app';
import { ref, watch, computed } from '@vue/composition-api';

import { useOrder } from 'shares/transaction/composables/useOrder.js';
import { usePayMethod } from './composables/usePayMethod.js';
import { useStoreInfo } from 'shares/transaction/composables/useStoreInfo.js';
import { usePayTrack } from './composables/usePayTrack.js';
import { useScroll } from './composables/useScroll.js';

export const usePay = () => {
  // 1. 生命周期管理（只在主 composable 中）
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

  // 2. 引入所有子 composables（纯逻辑，无生命周期）
  const { orderReviewData, fetchReviewData } = useOrder();
  const { selectedPayMethod, payMethodList, submitOrder } = usePayMethod();
  const { isNextDayOrder, currentStore } = useStoreInfo();
  const { trackPayResult, trackPaymentToolAction } = usePayTrack();
  const { isShow: isToOrderShow, scrollToTarget: scrollToOrder } = useScroll();

  // 3. 跨 composables 的 watch 管理（只在主 composable 中）
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal === PaymentType.SVC ? '星礼卡' : '微信');
    }
  });

  watch(orderReviewData, (newVal) => {
    if (newVal) {
      fetchPayMethod(newVal.channel, newVal.storeId, newVal.payPrice);
    }
  });

  // 4. 组合计算属性（依赖多个子 composables）
  const canSubmit = computed(() => {
    return isLogin.value &&
           orderReviewData.value &&
           selectedPayMethod.value &&
           !isDisabledSubmit.value;
  });

  // 5. 组合方法
  const handleSubmit = async () => {
    if (!checkAgreementCheck()) return;
    await submitOrder();
  };

  // 6. 返回所有需要的状态和方法
  return {
    orderReviewData, selectedPayMethod, payMethodList,
    isNextDayOrder, currentStore, isToOrderShow,
    canSubmit, handleSubmit, scrollToOrder, fetchReviewData,
  };
};
```

### 子 Composable：纯业务逻辑

**职责**：处理单一业务逻辑，无生命周期

**特点**：
- 不包含 `onLoad`, `onShow` 等生命周期
- 只包含 `ref`, `computed`, `methods`
- 可以依赖其他子 composables
- 可以依赖 store

#### 数据层子 Composable

```javascript
// useOrder.js - 纯数据逻辑
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

#### 依赖其他子 Composable

```javascript
// usePayMethod.js - 依赖其他子 composables
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

## Watch 管理规范

### 跨 Composables 的 Watch（在主 Composable 中管理）

```javascript
// usePay.js
export const usePay = () => {
  const { selectedPayMethod } = usePayMethod();
  const { trackPaymentToolAction } = usePayTrack();
  const { orderReviewData } = useOrder();
  const { fetchPayMethod } = usePayMethod();

  // 跨 composables 的 watch 只在主 composable 中管理
  watch(selectedPayMethod, (newVal, oldVal) => {
    if (newVal !== oldVal) {
      trackPaymentToolAction(newVal === PaymentType.SVC ? '星礼卡' : '微信');
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

### Composable 内部的 Watch（在子 Composable 中管理）

```javascript
// useOrder.js - 子 composable 管理内部的 watch
export const useOrder = () => {
  const orderReviewData = ref(null);

  // 内部的 watch 在子 composable 中管理
  watch(orderReviewData, (newVal) => {
    if (newVal) {
      console.log('Order review data updated:', newVal);
    }
  });

  return { orderReviewData };
};
```

## 错误示例

```javascript
// 错误：子 composable 包含生命周期
export function useOrderData() {
  onLoad(() => { fetchData(); }); // 严禁！
  return { data };
}

// 错误：Default export
export default function useOrderDetail() {}

// 错误：非响应式
export function useCounter() {
  let count = 0; // 应该用 ref
  return { count };
}

// 错误：直接修改 props
export function useProps(props) {
  props.value = 'new value'; // 不要直接修改
}

// 错误：职责不清
export function useEverything() {
  // 数据获取、业务逻辑、埋点、路由全部混在一起
}
```

## 正确示例

```javascript
// 正确：主 composable 管理生命周期
export function usePay() {
  onLoad(() => { /* 初始化 */ });
  onShow(() => { /* 刷新 */ });

  const { orderReviewData } = useOrder();
  const { selectedPayMethod } = usePayMethod();

  watch(selectedPayMethod, (newVal) => {
    trackPaymentToolAction(newVal);
  });

  return { orderReviewData, selectedPayMethod };
}

// 正确：子 composable 纯逻辑
export function useOrder() {
  const orderReviewData = ref(null);
  const fetchReviewData = async (params) => {
    orderReviewData.value = await orderReview(params);
  };
  return { orderReviewData, fetchReviewData };
}

// 正确：不修改 props，使用本地 ref
export function useProps(props) {
  const localValue = ref(props.value);
  return { localValue };
}

// 正确：单一职责
export function useOrderData() { /* 只负责数据获取 */ }
export function useOrderLogic() { /* 只负责业务逻辑 */ }
```

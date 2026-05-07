# 听云监控完整 API 指南

## 核心 API

### 1. 获取监控实例

```javascript
import { getMonitorService } from 'tingyun/index.js';

// 异步获取监控实例
getMonitorService().then((service) => {
    console.log('异常监控实例已获取', service);

    // 设置全局上下文
    service.setContext({
        userId: '12345',
        version: '1.0.0'
    });
});
```

**实现类**: `src/shares/common/sdk/monitor/base/TingyunMonitor.js`

| 方法 | 说明 |
|------|------|
| `setContext(context)` | 设置全局异常监控上下文信息 |
| `report(data)` | 上报异常监控信息 |
| `reportError(error)` | 上报错误异常监控信息（基于 report 封装） |

### 2. 使用 Hook 便捷上报

```javascript
import { useMonitorReport } from 'tingyun/index.js';

export default {
    setup() {
        const monitorReport = useMonitorReport();

        const handleError = () => {
            monitorReport.error({
                name: 'OrderSubmitError',
                message: '订单提交失败',
                orderId: '123456',
                errorCode: 500
            });
        };

        return { handleError };
    }
};
```

## 日志分级

| 级别 | 方法 | 使用场景 |
|------|------|---------|
| `error` | `monitorReport.error()` | 系统错误、异常捕获 |
| `warn` | `monitorReport.warn()` | 警告信息、潜在问题 |
| `info` | `monitorReport.info()` | 业务关键信息 |
| `debug` | `monitorReport.debug()` | 调试信息（开发环境） |
| `log` | `monitorReport.log()` | 普通日志 |

### 分级上报示例

```javascript
// 定义异常类型枚举
export const ExceptionType = {
    ORDER_SUBMIT_ERROR: 'orderSubmitError',
    PAYMENT_ERROR: 'paymentError',
    COUPON_ERROR: 'couponError',
    NETWORK_ERROR: 'networkError'
};

// 业务代码中使用
const submitOrder = async () => {
    try {
        monitorReport.info({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: '开始提交订单',
            orderId: orderInfo.id
        });

        const result = await api.submitOrder(orderInfo);

        monitorReport.info({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: '订单提交成功',
            orderId: orderInfo.id,
            result
        });

    } catch (error) {
        monitorReport.error({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: '订单提交失败',
            orderId: orderInfo.id,
            error: error.message,
            stack: error.stack
        });

        throw error;
    }
};
```

## 错误捕获

```javascript
// 捕获异常并自动上报
monitorReport.captureError(new Error('测试异常'));

// 捕获异步异常
try {
    await riskyOperation();
} catch (error) {
    monitorReport.captureError(error);
}

// 在 Vue 组件中捕获
export default {
    methods: {
        async handleClick() {
            try {
                await this.doSomething();
            } catch (error) {
                this.$monitorReport.captureError(error, {
                    component: 'MyComponent',
                    action: 'handleClick'
                });
            }
        }
    }
};
```

## 业务监控 Hook 封装

```javascript
// src/composables/useBusinessMonitor.js
import { useMonitorReport } from 'tingyun/index.js';
import { ExceptionType } from '@/constants/exceptionTypes.js';

export function useBusinessMonitor() {
    const monitorReport = useMonitorReport();

    /**
     * 监控 API 调用
     */
    const monitorApiCall = async (apiName, apiFunction, params = {}) => {
        const startTime = Date.now();

        try {
            monitorReport.info({
                name: `api:${apiName}:start`,
                message: `开始调用 ${apiName}`,
                params
            });

            const result = await apiFunction(params);
            const duration = Date.now() - startTime;

            monitorReport.info({
                name: `api:${apiName}:success`,
                message: `${apiName} 调用成功`,
                duration,
                result
            });

            return result;

        } catch (error) {
            const duration = Date.now() - startTime;

            monitorReport.error({
                name: `api:${apiName}:error`,
                message: `${apiName} 调用失败`,
                duration,
                error: error.message,
                stack: error.stack
            });

            throw error;
        }
    };

    /**
     * 监控用户行为
     */
    const trackUserAction = (action, params = {}) => {
        monitorReport.log({
            name: `user:action:${action}`,
            message: `用户执行 ${action}`,
            action,
            params
        });
    };

    /**
     * 监控页面性能
     */
    const trackPagePerformance = (pageName, metrics = {}) => {
        monitorReport.info({
            name: `page:performance:${pageName}`,
            message: `页面 ${pageName} 性能数据`,
            pageName,
            metrics
        });
    };

    return { monitorApiCall, trackUserAction, trackPagePerformance, monitorReport };
}
```

## 统一异常类型定义

```javascript
// src/constants/exceptionTypes.js
export const ExceptionType = {
    // 订单模块
    ORDER_SUBMIT_ERROR: 'order:submitError',
    ORDER_PAY_ERROR: 'order:payError',
    ORDER_CANCEL_ERROR: 'order:cancelError',

    // 支付模块
    PAYMENT_INIT_ERROR: 'payment:initError',
    PAYMENT_CONFIRM_ERROR: 'payment:confirmError',

    // 优惠券模块
    COUPON_FETCH_ERROR: 'coupon:fetchError',
    COUPON_APPLY_ERROR: 'coupon:applyError',

    // 网络模块
    NETWORK_TIMEOUT: 'network:timeout',
    NETWORK_OFFLINE: 'network:offline',

    // 系统模块
    SYSTEM_STORAGE_ERROR: 'system:storageError',
    SYSTEM_LOCATION_ERROR: 'system:locationError'
};
```

## 相关文件

| 文件 | 说明 |
|------|------|
| `tingyun/index.js` | SDK 入口，提供 `getMonitorService` 和 `useMonitorReport` |
| `src/shares/common/sdk/monitor/base/TingyunMonitor.js` | 监控核心实现类 |

## 注意事项

1. **异步初始化**：`getMonitorService()` 返回 Promise，确保调用前已初始化完成
2. **错误处理**：监控上报不应阻塞业务流程，建议用 try-catch 包裹
3. **性能考虑**：高频操作（如滚动、点击）不要频繁上报，需要节流
4. **隐私合规**：上报数据不要包含敏感信息（如密码、手机号）
5. **生产环境**：开发环境可开启详细日志，生产环境建议只上报 error 级别

## 快速开始

```javascript
// 1. 引入 Hook
import { useMonitorReport } from 'tingyun/index.js';

// 2. 在组件中使用
export default {
    setup() {
        const monitorReport = useMonitorReport();

        const handleError = (error) => {
            // 3. 上报错误
            monitorReport.error({
                name: 'BusinessError',
                message: error.message,
                stack: error.stack
            });
        };

        return { handleError };
    }
};
```

# Tingyun Monitoring Full API Guide

## Core API

### 1. Get Monitoring Instance

```javascript
import { getMonitorService } from 'tingyun/index.js';

// Async get monitoring instance
getMonitorService().then((service) => {
    console.log('Monitoring instance acquired', service);

    // Set global context
    service.setContext({
        userId: '12345',
        version: '1.0.0'
    });
});
```

**Implementation Class**: `src/shares/common/sdk/monitor/base/TingyunMonitor.js`

| Method | Description |
|--------|-------------|
| `setContext(context)` | Set global exception monitoring context info |
| `report(data)` | Report exception monitoring info |
| `reportError(error)` | Report error exception monitoring info (wrapped around report) |

### 2. Use Hook for Convenient Reporting

```javascript
import { useMonitorReport } from 'tingyun/index.js';

export default {
    setup() {
        const monitorReport = useMonitorReport();

        const handleError = () => {
            monitorReport.error({
                name: 'OrderSubmitError',
                message: 'Order submission failed',
                orderId: '123456',
                errorCode: 500
            });
        };

        return { handleError };
    }
};
```

## Log Levels

| Level | Method | Use Case |
|-------|--------|----------|
| `error` | `monitorReport.error()` | System errors, exception capture |
| `warn` | `monitorReport.warn()` | Warnings, potential issues |
| `info` | `monitorReport.info()` | Key business information |
| `debug` | `monitorReport.debug()` | Debug info (development only) |
| `log` | `monitorReport.log()` | General logging |

### Leveled Reporting Example

```javascript
// Define exception type enum
export const ExceptionType = {
    ORDER_SUBMIT_ERROR: 'orderSubmitError',
    PAYMENT_ERROR: 'paymentError',
    COUPON_ERROR: 'couponError',
    NETWORK_ERROR: 'networkError'
};

// Usage in business code
const submitOrder = async () => {
    try {
        monitorReport.info({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: 'Starting order submission',
            orderId: orderInfo.id
        });

        const result = await api.submitOrder(orderInfo);

        monitorReport.info({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: 'Order submitted successfully',
            orderId: orderInfo.id,
            result
        });

    } catch (error) {
        monitorReport.error({
            name: ExceptionType.ORDER_SUBMIT_ERROR,
            message: 'Order submission failed',
            orderId: orderInfo.id,
            error: error.message,
            stack: error.stack
        });

        throw error;
    }
};
```

## Error Capture

```javascript
// Capture exception and auto-report
monitorReport.captureError(new Error('Test exception'));

// Capture async exception
try {
    await riskyOperation();
} catch (error) {
    monitorReport.captureError(error);
}

// Capture in Vue component
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

## Business Monitoring Hook Pattern

```javascript
// src/composables/useBusinessMonitor.js
import { useMonitorReport } from 'tingyun/index.js';
import { ExceptionType } from '@/constants/exceptionTypes.js';

export function useBusinessMonitor() {
    const monitorReport = useMonitorReport();

    /**
     * Monitor API call
     */
    const monitorApiCall = async (apiName, apiFunction, params = {}) => {
        const startTime = Date.now();

        try {
            monitorReport.info({
                name: `api:${apiName}:start`,
                message: `Starting ${apiName} call`,
                params
            });

            const result = await apiFunction(params);
            const duration = Date.now() - startTime;

            monitorReport.info({
                name: `api:${apiName}:success`,
                message: `${apiName} call succeeded`,
                duration,
                result
            });

            return result;

        } catch (error) {
            const duration = Date.now() - startTime;

            monitorReport.error({
                name: `api:${apiName}:error`,
                message: `${apiName} call failed`,
                duration,
                error: error.message,
                stack: error.stack
            });

            throw error;
        }
    };

    /**
     * Monitor user action
     */
    const trackUserAction = (action, params = {}) => {
        monitorReport.log({
            name: `user:action:${action}`,
            message: `User performed ${action}`,
            action,
            params
        });
    };

    /**
     * Monitor page performance
     */
    const trackPagePerformance = (pageName, metrics = {}) => {
        monitorReport.info({
            name: `page:performance:${pageName}`,
            message: `Page ${pageName} performance data`,
            pageName,
            metrics
        });
    };

    return { monitorApiCall, trackUserAction, trackPagePerformance, monitorReport };
}
```

## Unified Exception Type Definitions

```javascript
// src/constants/exceptionTypes.js
export const ExceptionType = {
    // Order module
    ORDER_SUBMIT_ERROR: 'order:submitError',
    ORDER_PAY_ERROR: 'order:payError',
    ORDER_CANCEL_ERROR: 'order:cancelError',

    // Payment module
    PAYMENT_INIT_ERROR: 'payment:initError',
    PAYMENT_CONFIRM_ERROR: 'payment:confirmError',

    // Coupon module
    COUPON_FETCH_ERROR: 'coupon:fetchError',
    COUPON_APPLY_ERROR: 'coupon:applyError',

    // Network module
    NETWORK_TIMEOUT: 'network:timeout',
    NETWORK_OFFLINE: 'network:offline',

    // System module
    SYSTEM_STORAGE_ERROR: 'system:storageError',
    SYSTEM_LOCATION_ERROR: 'system:locationError'
};
```

## Related Files

| File | Description |
|------|-------------|
| `tingyun/index.js` | SDK entry, provides `getMonitorService` and `useMonitorReport` |
| `src/shares/common/sdk/monitor/base/TingyunMonitor.js` | Monitoring core implementation class |

## Notes

1. **Async initialization**: `getMonitorService()` returns a Promise, ensure initialization is complete before calling
2. **Error handling**: Monitoring reports should not block business flow, recommend wrapping in try-catch
3. **Performance considerations**: High-frequency operations (scrolling, clicking) should not report frequently, need throttling
4. **Privacy compliance**: Report data must not contain sensitive information (passwords, phone numbers)
5. **Production environment**: Detailed logs can be enabled in development, recommend only error-level reporting in production

## Quick Start

```javascript
// 1. Import Hook
import { useMonitorReport } from 'tingyun/index.js';

// 2. Use in component
export default {
    setup() {
        const monitorReport = useMonitorReport();

        const handleError = (error) => {
            // 3. Report error
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

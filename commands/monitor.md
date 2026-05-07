# /monitor - 接入监控上报

## 开发流程

### Step 1: 引入监控 Hook

```javascript
import { useMonitorReport } from 'tingyun/index.js';
```

### Step 2: 在组件中使用

```javascript
export default {
    setup() {
        const monitorReport = useMonitorReport();
        return { monitorReport };
    }
};
```

### Step 3: 按级别上报

| 级别 | 方法 | 场景 |
|------|------|------|
| error | `monitorReport.error()` | 系统错误、异常捕获 |
| warn | `monitorReport.warn()` | 警告信息、潜在问题 |
| info | `monitorReport.info()` | 业务关键信息 |
| debug | `monitorReport.debug()` | 调试信息（开发环境） |
| log | `monitorReport.log()` | 普通日志 |

### Step 4: 捕获异常

```javascript
try {
    await riskyOperation();
} catch (error) {
    monitorReport.error({
        name: 'OrderSubmitError',
        message: error.message,
        stack: error.stack
    });
}
```

## 关键约束

1. **上报不阻塞业务**：用 try-catch 包裹监控代码
2. **高频操作需节流**：滚动、点击等不要频繁上报
3. **不上报敏感信息**：禁止包含密码、手机号等
4. **生产环境控制级别**：建议只上报 error 级别

## 详细参考

→ 读取 [reference/monitor.md](../reference/monitor.md) 获取完整 API、业务监控 Hook 封装、异常类型定义

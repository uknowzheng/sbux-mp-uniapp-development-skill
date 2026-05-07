# /monitor - Integrate Monitoring & Reporting

## Development Workflow

### Step 1: Import Monitoring Hook

```javascript
import { useMonitorReport } from 'tingyun/index.js';
```

### Step 2: Use in Component

```javascript
export default {
    setup() {
        const monitorReport = useMonitorReport();
        return { monitorReport };
    }
};
```

### Step 3: Report by Level

| Level | Method | Scenario |
|-------|--------|----------|
| error | `monitorReport.error()` | System errors, exception capture |
| warn | `monitorReport.warn()` | Warnings, potential issues |
| info | `monitorReport.info()` | Key business information |
| debug | `monitorReport.debug()` | Debug info (development only) |
| log | `monitorReport.log()` | General logging |

### Step 4: Capture Exceptions

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

## Key Constraints

1. **Reporting must not block business**: Wrap monitoring code in try-catch
2. **Throttle high-frequency operations**: Don't report frequently for scrolling, clicking, etc.
3. **No sensitive information**: Never include passwords, phone numbers, etc.
4. **Control level in production**: Recommend only reporting error level in production

## Detailed Reference

→ Read [reference/monitor.md](../reference/monitor.md) for full API, business monitoring hook patterns, exception type definitions

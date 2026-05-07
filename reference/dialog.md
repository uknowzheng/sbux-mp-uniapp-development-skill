# 弹窗 Composables 使用指南

项目提供三个弹窗 Composables，统一管理各类弹窗交互。

| Composable | 组件 | 用途 |
|-----------|------|------|
| `useAlertDialog` | `uni-alert` | 确认/取消类弹窗 |
| `useSlideDialog` | `slide-dialog` | 信息展示类弹窗 |
| `usePromptDialog` | `uni-prompt` | 多按钮选择类弹窗 |

**核心优势**：一个组件多次复用 / 支持回调和 Promise 两种模式 / 状态自动管理

## useAlertDialog

### 引入与初始化

```javascript
import { useAlertDialog } from 'common/composables/useAlertDialog.js';

const { alertState, showAlert, handleAlertConfirm, handleAlertCancel } = useAlertDialog();
```

### 模板中定义组件（只需一次）

```vue
<uni-alert
    v-model="alertState.value"
    :title="alertState.title"
    :content="alertState.content"
    :cancel-enable="alertState.cancelEnable"
    :cancel-text="alertState.cancelText"
    :confirm-text="alertState.confirmText"
    @confirm="handleAlertConfirm"
    @cancel="handleAlertCancel"
/>
```

### showAlert(options)

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| title | String/Boolean | false | 弹窗标题 |
| content | String/Boolean | true | 弹窗内容 |
| cancelEnable | Boolean | true | 是否显示取消按钮 |
| cancelText | String | '取消' | 取消按钮文案 |
| confirmText | String | '确认' | 确认按钮文案 |
| onConfirm | Function | - | 确认回调（回调模式） |
| onCancel | Function | - | 取消回调（回调模式） |

**返回值**：
- 回调模式（传入 onConfirm/onCancel）：无返回值
- Promise 模式（不传回调）：返回 `Promise<Boolean>`

### 使用场景

```javascript
// 基础提示
showAlert({ title: '提示', content: '操作成功', cancelEnable: false });

// 删除确认（回调模式）
showAlert({
    title: '删除确认',
    content: '确定要删除吗？',
    onConfirm: () => { /* 执行删除 */ },
});

// Promise 模式
const confirmed = await showAlert({ title: '清空购物车', content: '确定要清空吗？', cancelEnable: true });
if (confirmed) { /* 执行清空 */ }
```

## useSlideDialog

### 引入与初始化

```javascript
import { useSlideDialog } from 'common/composables/useSlideDialog.js';

const { dialogState, showDialog } = useSlideDialog();
```

### 模板中定义组件

```vue
<slide-dialog
    v-model="dialogState.value"
    :title="dialogState.title"
    :content="dialogState.content"
    :richText="dialogState.richText"
/>
```

### showDialog(options)

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| title | String | '' | 弹窗标题 |
| content | String | '' | 弹窗内容 |
| richText | Boolean | false | 是否为富文本 |
| iconClose | Boolean | true | 是否显示关闭图标 |
| tapToClose | Boolean | false | 点击内容区域是否关闭 |
| type | String | 'bottom' | 弹出位置 |
| contentHeight | Number | 500 | 内容区域高度（rpx） |
| maskClick | Boolean | true | 点击遮罩是否关闭 |

```javascript
// 展示用户协议
showDialog({ title: '用户协议', content: '协议内容...' });

// 展示富文本活动规则
showDialog({ title: '活动规则', content: '<p>...</p>', richText: true });
```

## usePromptDialog

### 引入与初始化

```javascript
import { usePromptDialog } from 'common/composables/usePromptDialog.js';

const { promptState, showPrompt, handlePromptClick } = usePromptDialog();
```

### 模板中定义组件

```vue
<uni-prompt
    v-model="promptState.value"
    :title="promptState.title"
    :content="promptState.content"
    :buttons="promptState.buttons"
    @click="handlePromptClick"
/>
```

### showPrompt(options)

| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| title | String/Boolean | false | 弹窗标题 |
| content | String/Boolean | false | 弹窗内容 |
| buttons | Array | [] | 按钮配置数组 |
| type | String | 'center' | 弹出位置 |
| maskClick | Boolean | false | 点击遮罩是否关闭 |
| onClick | Function | - | 按钮点击回调（回调模式） |

**buttons 配置**：`{ text: '按钮文案', type: 'cancel' }`

**返回值**：
- 回调模式（传入 onClick）：无返回值
- Promise 模式（不传 onClick）：返回 `Promise<{ button, index }>`

```javascript
// 两按钮选择（回调模式）
showPrompt({
    title: '提示',
    content: '确定执行吗？',
    buttons: [{ text: '取消', type: 'cancel' }, { text: '确认', type: 'confirm' }],
    onClick: (button) => { if (button.type === 'confirm') { /* ... */ } },
});

// Promise 模式
const { button } = await showPrompt({
    title: '选择支付方式',
    buttons: [{ text: '微信支付', type: 'wechat' }, { text: '取消', type: 'cancel' }],
});
if (button.type === 'wechat') { /* ... */ }
```

## 常见问题

### 同一页面多业务场景会串吗？

不会。只要业务场景是串行的（用户操作完一个弹窗再操作下一个），同一个组件会根据调用时传入的配置显示不同内容。

### 需要同时显示多个弹窗？

创建多个 hook 实例，模板中定义多个组件：

```javascript
const deleteAlert = useAlertDialog();
const saveAlert = useAlertDialog();
```

```vue
<uni-alert v-model="deleteAlertState.value" ... />
<uni-alert v-model="saveAlertState.value" ... />
```

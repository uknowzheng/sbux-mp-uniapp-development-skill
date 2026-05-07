# Dialog Composables Usage Guide

The project provides three Dialog Composables for unified management of dialog interactions.

| Composable | Component | Usage |
|-----------|-----------|-------|
| `useAlertDialog` | `uni-alert` | Confirm/cancel dialogs |
| `useSlideDialog` | `slide-dialog` | Information display dialogs |
| `usePromptDialog` | `uni-prompt` | Multi-button selection dialogs |

**Core Advantages**: One component reused multiple times / Supports both callback and Promise modes / Automatic state management

## useAlertDialog

### Import and Initialize

```javascript
import { useAlertDialog } from 'common/composables/useAlertDialog.js';

const { alertState, showAlert, handleAlertConfirm, handleAlertCancel } = useAlertDialog();
```

### Define Component in Template (Only Once)

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

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| title | String/Boolean | false | Dialog title |
| content | String/Boolean | true | Dialog content |
| cancelEnable | Boolean | true | Whether to show cancel button |
| cancelText | String | 'Cancel' | Cancel button text |
| confirmText | String | 'Confirm' | Confirm button text |
| onConfirm | Function | - | Confirm callback (callback mode) |
| onCancel | Function | - | Cancel callback (callback mode) |

**Return Value**:
- Callback mode (pass onConfirm/onCancel): No return value
- Promise mode (no callbacks passed): Returns `Promise<Boolean>`

### Usage Examples

```javascript
// Basic notification
showAlert({ title: 'Notice', content: 'Operation successful', cancelEnable: false });

// Delete confirmation (callback mode)
showAlert({
    title: 'Delete Confirmation',
    content: 'Are you sure you want to delete?',
    onConfirm: () => { /* Execute delete */ },
});

// Promise mode
const confirmed = await showAlert({ title: 'Clear Cart', content: 'Are you sure?', cancelEnable: true });
if (confirmed) { /* Execute clear */ }
```

## useSlideDialog

### Import and Initialize

```javascript
import { useSlideDialog } from 'common/composables/useSlideDialog.js';

const { dialogState, showDialog } = useSlideDialog();
```

### Define Component in Template

```vue
<slide-dialog
    v-model="dialogState.value"
    :title="dialogState.title"
    :content="dialogState.content"
    :richText="dialogState.richText"
/>
```

### showDialog(options)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| title | String | '' | Dialog title |
| content | String | '' | Dialog content |
| richText | Boolean | false | Whether content is rich text |
| iconClose | Boolean | true | Whether to show close icon |
| tapToClose | Boolean | false | Whether tapping content area closes dialog |
| type | String | 'bottom' | Popup position |
| contentHeight | Number | 500 | Content area height (rpx) |
| maskClick | Boolean | true | Whether clicking mask closes dialog |

```javascript
// Show user agreement
showDialog({ title: 'User Agreement', content: 'Agreement content...' });

// Show rich text activity rules
showDialog({ title: 'Activity Rules', content: '<p>...</p>', richText: true });
```

## usePromptDialog

### Import and Initialize

```javascript
import { usePromptDialog } from 'common/composables/usePromptDialog.js';

const { promptState, showPrompt, handlePromptClick } = usePromptDialog();
```

### Define Component in Template

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

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| title | String/Boolean | false | Dialog title |
| content | String/Boolean | false | Dialog content |
| buttons | Array | [] | Button configuration array |
| type | String | 'center' | Popup position |
| maskClick | Boolean | false | Whether clicking mask closes dialog |
| onClick | Function | - | Button click callback (callback mode) |

**buttons Configuration**: `{ text: 'Button Text', type: 'cancel' }`

**Return Value**:
- Callback mode (pass onClick): No return value
- Promise mode (no onClick passed): Returns `Promise<{ button, index }>`

```javascript
// Two-button selection (callback mode)
showPrompt({
    title: 'Notice',
    content: 'Are you sure?',
    buttons: [{ text: 'Cancel', type: 'cancel' }, { text: 'Confirm', type: 'confirm' }],
    onClick: (button) => { if (button.type === 'confirm') { /* ... */ } },
});

// Promise mode
const { button } = await showPrompt({
    title: 'Select Payment Method',
    buttons: [{ text: 'WeChat Pay', type: 'wechat' }, { text: 'Cancel', type: 'cancel' }],
});
if (button.type === 'wechat') { /* ... */ }
```

## FAQ

### Will multiple business scenarios on the same page conflict?

No. As long as business scenarios are sequential (user finishes one dialog before the next), the same component will display different content based on the configuration passed at call time.

### Need to show multiple dialogs simultaneously?

Create multiple hook instances and define multiple components in the template:

```javascript
const deleteAlert = useAlertDialog();
const saveAlert = useAlertDialog();
```

```vue
<uni-alert v-model="deleteAlertState.value" ... />
<uni-alert v-model="saveAlertState.value" ... />
```

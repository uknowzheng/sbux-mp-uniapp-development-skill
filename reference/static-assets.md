# 静态资源管理规范

## 目录结构

```bash
src/static/
├── shares/                    # 业务包静态资源
│   ├── public/               # 公共资源
│   │   └── images/
│   │       ├── reservation/  # 预约相关
│   │       ├── mod/          # 专星送相关
│   │       ├── mop/          # 啡快相关
│   │       └── ctn/          # 定制化相关
│   ├── common/               # 通用资源
│   ├── activity/             # 活动包资源
│   └── ...                   # 其他业务包
```

## 核心原则

1. 按业务包组织资源
2. 使用 `imgUrl()` 方法引用图片
3. 编译时自动处理路径拼接

## imgUrl 使用方式

在 Vue 模板中引用图片时，**必须**使用 `imgUrl()` 方法。

### 动态绑定（:src）

```vue
<image :src="imgUrl('shares/public/images/reservation/select-no-city.png')" />
```

### 静态绑定（src）

```vue
<image src="imgUrl('shares/public/images/mod/banner.png')" />
```

### 完整示例

```vue
<template>
    <view>
        <!-- 动态绑定 -->
        <image :src="imgUrl('shares/public/images/reservation/select-no-city.png')" />

        <!-- 静态绑定 -->
        <image src="imgUrl('shares/public/images/mod/banner.png')" />

        <!-- 带其他属性 -->
        <image class="icon" :src="imgUrl('shares/public/images/mop/choiced.png')" mode="aspectFit" />

        <!-- 完整 URL（保持不变） -->
        <image :src="imgUrl('https://example.com/image.png')" />
    </view>
</template>
```

## 路径说明

| 路径类型 | 写法 | 处理方式 |
|---------|------|---------|
| 相对路径 | `imgUrl('shares/common/icon.png')` | 自动添加 `/static/` 前缀 |
| 完整 URL | `imgUrl('https://example.com/image.png')` | 保持不变 |

## 注意事项

1. 只能在 `<image>` 标签中使用 `imgUrl()`
2. 必须使用单引号包裹路径：`imgUrl('path')`
3. 编译时自动转换为实际路径，无需在 `setup()` 中导入 `imgUrl` 函数
4. 仅支持 `imgUrl()` 方式，不要使用 `$imgUrl()`

## 图片命名规范

- 使用小写字母和连字符：`user-avatar.png`
- 避免中文和特殊字符
- 语义化命名：`icon-close.png`、`bg-header.png`

## 性能优化

- 使用 WebP 格式（兼容性允许的情况下）
- 压缩图片后再上传
- 懒加载大图片

## 相关文件

| 文件 | 说明 |
|------|------|
| `plugins/imgUrl-loader.js` | Loader 实现 |
| `vue.config.js` | 构建配置 |
| `plugins/ImgUrlCompilePlugin.md` | 使用文档 |

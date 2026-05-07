# Static Asset Management Guide

## Directory Structure

```bash
src/static/
├── shares/                    # Business package static assets
│   ├── public/               # Common assets
│   │   └── images/
│   │       ├── reservation/  # Reservation-related
│   │       ├── mod/          # MOD delivery-related
│   │       ├── mop/          # MOP pickup-related
│   │       └── ctn/          # Customization-related
│   ├── common/               # Shared assets
│   ├── activity/             # Activity package assets
│   └── ...                   # Other business packages
```

## Core Principles

1. Organize assets by business package
2. Use `imgUrl()` method to reference images
3. Path concatenation is handled automatically at build time

## imgUrl Usage

When referencing images in Vue templates, you **must** use the `imgUrl()` method.

### Dynamic Binding (:src)

```vue
<image :src="imgUrl('shares/public/images/reservation/select-no-city.png')" />
```

### Static Binding (src)

```vue
<image src="imgUrl('shares/public/images/mod/banner.png')" />
```

### Complete Example

```vue
<template>
    <view>
        <!-- Dynamic binding -->
        <image :src="imgUrl('shares/public/images/reservation/select-no-city.png')" />

        <!-- Static binding -->
        <image src="imgUrl('shares/public/images/mod/banner.png')" />

        <!-- With other attributes -->
        <image class="icon" :src="imgUrl('shares/public/images/mop/choiced.png')" mode="aspectFit" />

        <!-- Full URL (remains unchanged) -->
        <image :src="imgUrl('https://example.com/image.png')" />
    </view>
</template>
```

## Path Explanation

| Path Type | Pattern | Processing |
|-----------|---------|-----------|
| Relative path | `imgUrl('shares/common/icon.png')` | Auto-prepends `/static/` prefix |
| Full URL | `imgUrl('https://example.com/image.png')` | Remains unchanged |

## Notes

1. Only use `imgUrl()` in `<image>` tags
2. Must use single quotes for paths: `imgUrl('path')`
3. Automatically converts to actual paths at build time, no need to import `imgUrl` function in `setup()`
4. Only `imgUrl()` is supported, do not use `$imgUrl()`

## Image Naming Conventions

- Use lowercase letters and hyphens: `user-avatar.png`
- Avoid Chinese and special characters
- Semantic naming: `icon-close.png`, `bg-header.png`

## Performance Optimization

- Use WebP format (where compatibility allows)
- Compress images before uploading
- Lazy-load large images

## Related Files

| File | Description |
|------|-------------|
| `plugins/imgUrl-loader.js` | Loader implementation |
| `vue.config.js` | Build configuration |
| `plugins/ImgUrlCompilePlugin.md` | Usage documentation |
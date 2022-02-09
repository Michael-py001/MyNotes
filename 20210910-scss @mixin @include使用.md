## scss @mixin @include使用

```scss
// 苹果安全底部高度
@mixin saveBottom($default:'0rpx') {
  padding-bottom: calc(#{$default} + #{env(safe-area-inset-bottom)});
}
```


# 20240428-使用CSS Module方式修改第三方组件样式

```css
.EditInput {
    .align-right {
        text-align: right;
        :global {
            .ant-input-number-input-wrap .ant-input-number-input {
                text-align: right;
            }
        }
    }
    .align-left {
        text-align: left;
    }
    .align-center {
        text-align: center;
    }
}

```

在tsx组件中使用css module方式代替`:v-deep()`，修改第三方组件样式
# 20230328-cssModule的使用

## Vue3 + TSX中使用cssModule

使用的后缀需要是: `xxx.module.less`，

使用语法：

`<div className={styles.类名}></div>`

```jsx
import styles from './styles/personDetail.module.less';

export default defineComponent({
  setup(props, ctx) {
      return () => {
          (
              <>
              	<div className={styles.personDetail}>hello world</div>
              </>
          )
      }
  }
})
```

```less
// personDetail.module.less
.personDetail {
  :global(.ant-btn) {
    margin-right: 15px;
    border-radius: 3px;
  }
  .btns {
    margin-top: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    box-sizing: border-box;
  }
}
```



## 全局作用域:global()的使用

如果在使用UI组件库时，要修改其样式，则需要用到:global()语法。括号里可以加上指定的类名，也可以直接在大括号里写其他类名。

```less
:global(.ant-btn) {
    margin-right: 15px;
    border-radius: 3px;
}
.global {
    .title {}
}
```

在global前面还可以加上属于哪个类名之下，避免污染其他页面的样式：

```less
.persion :global(.ant-btn) {
    margin-right: 15px;
    border-radius: 3px;
    .title {
        
    }
}
```

使用global后，就可以直接在div上写类名字符串了

```jsx
<div className={styles.persion}>
    	<h1 className="title">hello world</h1>
 </div>
```

## **composes** 样式复用

```css
.className {
  background-color: blue;
}

.title {
  composes: className; //继承className
  color: red;
}
```


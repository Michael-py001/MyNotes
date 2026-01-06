# 20251125-flex-grow:1与height:0实现自适应高度

`flex-grow: 1` 和 `height: 0` 放在一起用，是 **flex 布局中非常经典的“自适应填充”写法**。

下面我用最通俗、最直观的方式解释它们组合后的效果：

------

## ✅ **两者合起来的效果：让元素在纵向（column）或横向（row）中“自动撑满剩余空间”**

核心机制：

**1. `height: 0` → 把元素的“内容高度”压缩为 0**

意思是：
 这个元素本身不凭内容撑开高度，它的高度可以完全由 flex 布局分配。

**2. `flex-grow: 1` → 元素会“抢占”剩余空间**

如果父容器是 flex 布局（尤其是 `flex-direction: column`），
 这个元素会按照 grow=1 的比例去分配父容器的剩余空间。

------

## 📌 **组合后的实际效果：**

👉 **让这个元素在父容器中，自动填满剩余的空间（高度或宽度）！**

尤其常见于：

**场景例子：让中间内容区填满屏幕剩余高度**

比如一个布局：

```
头部（固定高度）
内容区（自适应剩余空间）
底部（固定高度）
```

内容区设置：

```css
.content {
  flex-grow: 1;
  height: 0;
  overflow: auto; /* 防止内容撑爆 */
}
```

效果👇：

- 头部固定高度
- 底部固定高度
- **中间 content 自动撑满剩余高度**
- 内容过长则内容区内部滚动，而不会把整个页面顶开

------

## 📘 为什么不用 `flex: 1` ？

因为：

```css
flex: 1;
```

等价于：

```css
flex: 1 1 0;
```

它内部其实包含了 `flex-basis: 0`，
 作用与 `height: 0` 在 column 布局下的效果一致。

但在某些浏览器或特殊布局下
 `height:0 + flex-grow:1` 更稳定，尤其是存在 `overflow:auto` 时。

------

## 🔍 总结

**`flex-grow:1; height:0;` 的组合让 flex 子项无视内容高度，按照比例占满父级剩余空间，非常适用于“中间自适应区域”的布局。**

## Demo 页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>flex-grow + height:0 Demo</title>
  <style>
    body {
      margin: 0;
      height: 100vh;
      display: flex;
      flex-direction: column;
      font-family: Arial, sans-serif;
    }

    header {
      background: #4a90e2;
      color: white;
      padding: 16px;
    }

    .content {
      flex-grow: 1;
      height: 0;
      overflow: auto;
      background: #f5f5f5;
      padding: 16px;
    }

    footer {
      background: #333;
      color: white;
      padding: 16px;
    }

    .box {
      background: white;
      border-radius: 8px;
      padding: 12px;
      margin-bottom: 16px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }
  </style>
</head>
<body>
  <header>顶部固定区域</header>

  <div class="content">
    <div class="box">可滚动内容 1</div>
    <div class="box">可滚动内容 2</div>
    <div class="box">可滚动内容 3</div>
    <div class="box">可滚动内容 4</div>
    <div class="box">可滚动内容 5</div>
    <div class="box">可滚动内容 6</div>
    <div class="box">可滚动内容 7</div>
    <div class="box">可滚动内容 8</div>
    <div class="box">可滚动内容 9</div>
    <div class="box">可滚动内容 10</div>
  </div>

  <footer>底部固定区域</footer>
</body>
</html>

```

![image-20251125092156024](https://s2.loli.net/2025/11/25/iEyOgLFmfkxrGa6.png)
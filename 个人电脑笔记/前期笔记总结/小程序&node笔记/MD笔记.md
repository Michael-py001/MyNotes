## 移动端页面设置

```javascript
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=0">
```

可以让页面内容按比例放大

## JS中要注意封装功能代码函数，减少全局变量。

## 居中小技巧

居中小技巧：

1. 水平居中：text-align:center；

2. 垂直居中：light-height; 需要先设置高度;

3. 水平居中：设置width百分百，再设置left百分百

   ```css
   #order_msg {
       position: fixed ;
       background-color: rgba(61,61,128,.7);
       width: 80%; //设置width百分百
       height: 40px;
       text-align: center;
       line-height: 40px;
       top: 0;
       left: 10%; //再设置left百分百
   }
   ```

   

4. flex布局：

   Justify-content:center;
   	align-items:center;

## insertBefore的用法

```javascript
let insertedNode = parentNode.insertBefore(newNode, referenceNode);
```



## 参数

`newNode`：将要插入的节点

` referenceNode`：被参照的节点（即要插在该节点之前）

`insertedNode`：插入后的节点

` parentNode`：父节点

要从父节点插入新的节点，而且该父节点里面要有子节点。




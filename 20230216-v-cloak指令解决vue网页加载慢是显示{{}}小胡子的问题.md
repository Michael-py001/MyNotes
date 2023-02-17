# 20230216-v-cloak指令解决vue网页加载慢是显示{{}}小胡子的问题

## 使用

把这个指令直接加在属性上，不是写在class里面

```css
// css
[v-cloak] {
	display: none!important;
}
```

![image-20230216181632014](https://s2.loli.net/2023/02/16/aRgDTrGzZ6YKhEM.png)

## 使用前

![image-20230216182002209](https://s2.loli.net/2023/02/16/sL7GRydqTuiJ5rc.png)

## 使用后

![image-20230216181939144](https://s2.loli.net/2023/02/16/YuMrRoAXviE9V3Q.png)
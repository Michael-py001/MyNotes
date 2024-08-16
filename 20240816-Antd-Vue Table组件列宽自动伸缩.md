# 20240816-Antd-Vue Table组件列宽自动伸缩

## 列宽自动伸缩

只需要设置Table的Scroll属性，x设置为'max-content'，即可实现列宽自动伸缩，自动按最长的那一列设置宽度

![image-20240816150440630](https://s2.loli.net/2024/08/16/iXHW7AvxUOajYme.png)

![image-20240816150329228](https://s2.loli.net/2024/08/16/uym47kafqUYHMwB.png)

![image-20240816150418943](https://s2.loli.net/2024/08/16/ZBluMqd2H8wtYK9.png)

![image-20240816150719972](https://s2.loli.net/2024/08/16/l7KCv3e5Bz1FMZt.png)

![image-20240816164649741](https://s2.loli.net/2024/08/16/iOLUQ2kvxGPNrVt.png)

## 列宽自动伸缩+最大宽度+省略号

最大宽度的实现是给每个列设置最大宽度：

![image-20240816164630861](https://s2.loli.net/2024/08/16/1YHtkDgCBZEyFx3.png)

省略号实现：直接给column设置: `ellipsis: true,`

![image-20240816162358116](https://s2.loli.net/2024/08/16/NI79zekyJ8nAuLq.png)

![image-20240816162407638](https://s2.loli.net/2024/08/16/pLn5yaRODr1uAct.png)
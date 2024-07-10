# 20240625-antd-vue 在Table中展示总结栏

需求：

![image-20240625173327336](https://s2.loli.net/2024/06/25/AsrQMHSz4nmi2qe.png)

![image-20240625173334319](https://s2.loli.net/2024/06/25/jcilxY1dCnGAIX2.png)

![image-20240629172639575](https://s2.loli.net/2024/06/29/CIfrRU6Q8jo2pbH.png)

## 实现方案

原本使用antd-vue的总结栏功能，但是不太能完全符合要求，最终使用了头尾插入数据 + 自定义pagination方案

![image-20240710182407774](https://s2.loli.net/2024/07/10/F69K2AaslcNyJvW.png)

## 使用这个方案的原因

因为官方的总结栏只能支持在顶部或者底部，不能实现同时出现，所以只能自己插入一行数据用作显示总结栏。

因为table会有分页，使用pagination设置属性的方式，table内部会处理传入的dateSource，会导致数据出现错乱。

所以使用单独的pagination，这样会避免出现数据错乱问题。
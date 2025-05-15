# 接口类型报错: type '_Map<String, dynamic>' is not a subtype of type xxx

> `flutter: Error in authentication: type '_Map<String, dynamic>' is not a subtype of type 'LoginUserInfo?'`

遇到这种报错，是因为直接使用了接口返回的数据，没有进行数据转换，接口返回的data是`Map<String, dynamic>`类型，但是定义的响应类型是`LoginUserInfo`，在运行时会报错，编译时不会报错。

![image-20250506170657827](https://s2.loli.net/2025/05/06/OskZulz4GKIx1e8.png)

使用：
![image-20250506170842367](https://s2.loli.net/2025/05/06/2lDeBRZoEwTAi1h.png)

解决方案：

需要在API层使用model数据模型中定义的`fromJson`方法转换，确保data类型正确。

![image-20250506171458699](https://s2.loli.net/2025/05/06/wef8x23oGKudOIn.png)

![image-20250506171448034](https://s2.loli.net/2025/05/06/2eGzHlmWgS4vTur.png)

数据模型转换中也要判断是否为空值和类型是否正确，因为传入的json为`dynamic`类型。

在这处理不符合接口定义类型的错误。

![image-20250506174701344](https://s2.loli.net/2025/05/06/QSJL2UpHr8ZcDnT.png)
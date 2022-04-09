# 20220326-Electron禁用browerView缓存

此方案是`electron`独有的禁用方式，即`Electron`支持的`Chrome`命令行开关，是直接相当于控制chrome浏览器的设置一样，不涉及网页的各种禁用缓存技术。

## 关键代码

```js
import {
	app,
	protocol,
	BrowserWindow,
	screen
} from 'electron'
//...
app.commandLine.appendSwitch("--disable-http-cache");
//...
app.on('ready', createWindow)
```



## 效果

![image-20220326094509324](https://s2.loli.net/2022/03/26/6avkleNMTdpjmW1.png)
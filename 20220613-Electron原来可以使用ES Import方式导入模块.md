# 20220613-Electron原来可以使用ES Import方式导入模块



[Announcing TypeScript support in Electron | Electron (electronjs.org)](https://www.electronjs.org/blog/typescript)

现在Electron全版本都支持TS，所以可以直接使用import导入模块

```js
import { ipcRenderer  } from 'electron'
```

只是在vite中，本身不支持
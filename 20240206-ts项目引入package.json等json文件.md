# 20240206-ts项目引入package.json等json文件

如果直接导入package.json，会提示报错。

![image-20240206104908380](https://s2.loli.net/2024/02/06/O9oytdJbFCM1k6X.png)

在tsconfig.json中加入：

```json
"compilerOptions": {
	"resolveJsonModule":true
}
```

然后在tsconfig.json中添加后，又提示报错

![image-20240206105024586](https://s2.loli.net/2024/02/06/Ktb7yQcOZfzw9UF.png)

继续加上以下属性：

```json
"compilerOptions": {
    "resolveJsonModule":true
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
}

```

![image-20240206114511147](https://s2.loli.net/2024/02/06/piNIXrj4KYD3C9J.png)
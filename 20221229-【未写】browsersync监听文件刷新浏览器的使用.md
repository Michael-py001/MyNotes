# 20221229-【未写】browsersync监听文件刷新浏览器的使用

> [Browsersync 命令行用法 (bootcss.com)](https://browsersync.bootcss.com/docs/command-line)

项目使用的框架是sails.js，使用ejs模块引擎开发，由于不会自动刷新，故使用browsersync帮助刷新。

下面的这个命令需要在项目根目录执行，因为这个sailjs框架不会生成.html文件，我就直接设置监听`**/*`任何文件改动都触发更新了。

命令：

```bash
browser-sync start --proxy  "localhost:1337"  --files  "**/*"
```

解析：

- 设置代理`--proxy`是因为项目启动在了`localhost:1337`，需要用`browser-sync`代理这个域名
- `--files`是监听的文件目录，`**/*`是代表所有任何文件都匹配。
# 20250818-lint-staged启动失败提示找不到版本的问题

在某一天，团队成员添加了lint-staged后，我重新npm install了，发现提交还是有问题，而且这个问题很诡异，package.json已经有lint-staged 的版本，node_modules里面也安装上了，但是还是提示:No matching version found for undefined@lint-staged。

![image-20250818161659496](https://s2.loli.net/2025/08/18/RkfGsnWyAzHhetK.png)

经过一番折腾，发现还差一个命令：`npx husky init`，还差初始化husky这一步才行，在终端执行后，提交就正常了。

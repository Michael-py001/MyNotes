# 20220526-GitHub Copilot无法连接使用的问题

## 提示无法连接服务器

最进无法插件使用，提示: GitHub Copilot could not connect to server. Extension activation failed: "connect ECONNREFUSED 127.0.0.1:443"

提示说无法连接服务器，估计又是像github一样被墙了，修改Host是最简单有效的办法，但是要知道插件连接的服务器ip。

经过一番搜索，从以下的文章找到了ip地址：

[编程已经离不开 GitHub-copilot 了 - 左志华的个人空间 - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/zuozhihua/blog/5531456)

## 设置host

1. 在host文件中加入以下解析:

```
140.82.112.5 github.com
140.82.112.5 api.github.com
```

2. 之后刷新dns缓存

```
ipconfig/flushdns
```

3. 重新打开vscode，可以连接使用了
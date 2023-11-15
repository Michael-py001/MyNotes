# 20231017-Nuxt3生成SSG静态站点部署到Github Page全流程

默认打包成SSG静态站点，只需要一个命令即可：

`npm run generate`

打包后的代码文件会放在`dist`目录下。

下面开始介绍如何打包再部署到github page。



详细版本可以看官方文档：[如何部署至GitHub Pages？ - NuxtJS | Nuxt.js 中文网](https://www.nuxtjs.cn/faq/github-pages)

## 1. 修改资源目录

> [Nuxt Configuration Reference · Nuxt](https://nuxt.com/docs/api/configuration/nuxt-config#buildassetsdir)

默认js,css等文件生成会在`_nuxt`目录，这里我们改成`static`目录

```ts
export default defineNuxtConfig({
  app: {
    buildAssetsDir: "static",
  },
})
```

## 2. 修改baseURL

> [Nuxt Configuration Reference · Nuxt](https://nuxt.com/docs/api/configuration/nuxt-config#baseurl)

由于部署到github page上后，访问的路径都是`https:用户名.github.io/仓库名/`，这会使图片等资源路径不对，导致访问不到。

```
export default defineNuxtConfig({
  app: {
    buildAssetsDir: "static",
    baseURL: process.env.DEPLOY_ENV === "GH_PAGES" ? "/xiandairiyu-nuxt/" : "/",
  },
})
```

## 3. 配置打包命令

```json
// package.json
"scripts": {
	"generate:gh-pages": "cross-env DEPLOY_ENV=GH_PAGES nuxt generate",
}
```

这里需要用到`cross-env`，可以运行`npm install cross-env`进行安装。

baseURL会读取这个`DEPLOY_ENV`环境变量，运行完这个命令后下一步就可以把打包产物推送到github page了

## 4. 推送到github pages

现在2023年github paegs已经很好用了，只需要把代码推送到你设置的分支，默认会识别`gh-pages`，就会自动运行一个Action把代码部署。所以我们要做的就是把产物推送到指定分支即可，剩下的交给Action自动部署。

![image-20231017180813760](https://s2.loli.net/2023/10/17/rCQZYBFkIMneg4j.png)

### 安装push-dir

```
npm install push-dir -D
```

这个库的作用是把代码推送到指定分支

### 配置部署命令

```json
{
	"deploy": "push-dir --dir=dist --branch=gh-pages --cleanup",
}
```

- `push-dir`：这是命令行工具的名字。
- `--dir=dist`：这个选项指定了要推送的目录的路径，这里是 `dist` 目录。这通常是构建工具（如 webpack 或 parcel）生成的静态文件的目录。
- `--branch=gh-pages`：这个选项指定了要推送到的 git 分支的名字，这里是 `gh-pages` 分支。GitHub Pages 服务会自动将这个分支的内容作为网站的内容进行托管。
- `--cleanup`：这个选项指示 `push-dir` 在推送之前清理目标分支，也就是说，它会删除目标分支上的所有旧文件，然后再将新文件推送上去。

然后到仓库里就可以看到正在部署了：

![image-20231017181337475](https://s2.loli.net/2023/10/17/FlgYLt3MJVZdBDU.png)
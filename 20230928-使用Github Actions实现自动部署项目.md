# 20230928-使用Github Actions实现自动部署项目

## Github Actions介绍

GitHub Actions 是一种持续集成和持续交付 (CI/CD) 平台，可用于自动执行生成、测试和部署管道。 您可以创建工作流程来构建和测试存储库的每个拉取请求，或将合并的拉取请求部署到生产环境。

Github action主要依赖项目目录`.github/workflows`下的.yml文件，每一个文件代表一个action，在你的github项目仓库中-->Actions下会自动识别这些文件。

![image-20230928151716622](https://s2.loli.net/2023/09/28/RgO2cD691kEPLzG.png)

![image-20230928151847463](https://s2.loli.net/2023/09/28/fU41c5T8oBzplwq.png)

yml文件中的name字段会被github识别：

![image-20230928152019626](https://s2.loli.net/2023/09/28/QTmjOpNgJ8dbzL1.png)

## 工作流语法基本介绍

> 官方文档 [触发工作流程 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/triggering-a-workflow)
>
> [GitHub Actions 的工作流语法 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/workflow-syntax-for-github-actions)

- on : 使用 `on` 键指定触发工作流的事件。

  - 使用单个事件：`on: push`

  - 多个事件: `on: [push,fork]`

  - 筛选器：当在指定分支下有push操作时触发

    ```yml
    on:
      push:
        branches:
          - main
          - 'releases/**'
    ```

- job: 工作流运行由一个或多个 `jobs` 组成，默认情况下并行运行。
  每个作业在 `runs-on` 指定的运行器环境中运行。

```yml
name: Build and Deploy # 工作流的名称
on: # 使用 on 键指定触发工作流的事件。
  push:
    branches:
      - master
  #pull request时也触发
  pull_request:
    branches:
      - master
# 授予 GITHUB_TOKEN 的默认权限
permissions:
  contents: write # 处理存储库的内容 write 允许操作创建发布
```

### 具体核心步骤

以下步骤均为`steps`下的每一个动作。

### 1. 获取仓库源码

```yml
steps:
      # 检出代码
      - name: Checkout 🛎️
        uses: actions/checkout@v3
```

### 2. 安装nodejs

可以在strategy中定义版本变量。

> 关于strategy : [GitHub Actions 的工作流语法 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix)

```yml
strategy:
      matrix:
        node-version: [16.x]
```

```yml
- name: Use Node.js ${{ matrix.node-version }} # 步骤2
        uses: actions/setup-node@v3 # 作用：安装nodejs
        with:
          node-version: ${{ matrix.node-version }} # 版本
```

### 3. 安装pnpm

> 官方文档： [pnpm/action-setup: Install pnpm package manager (github.com)](https://github.com/pnpm/action-setup/tree/master)

此步骤适用于pnpm的项目，如果你使用的是普通npm，可以跳过该步骤。

```yml
- name: Setup pnpm
        uses: pnpm/action-setup@v2
        id: pnpm-install
        with:
          version: 8.7.0
          run_install: false
```

### 4. 缓存依赖项，加快依赖构建速度(可选)

> 官方文档： [缓存依赖项以加快工作流程 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/caching-dependencies-to-speed-up-workflows#managing-caches)

```yml
# 利用pnpm的lock file作为hash key，对node_modules进行缓存，加快依赖构建的速度；如果缓存未命中则正常地install安装依赖即可
  - name: Get pnpm store directory
    shell: bash
    run: |
      echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

  - name: Setup pnpm cache
    uses: actions/cache@v3
    id: pnpm-cache
    with:
      path: ${{ env.STORE_PATH }}
      key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
      restore-keys: |
        ${{ runner.os }}-pnpm-store-

  # 缓存未命中
  - if: ${{ steps.pnpm-cache.outputs.cache-hit != 'true' }}
    name: List the state of node modules
    continue-on-error: true
    run: pnpm list
```

### 5. 安装依赖和部署

```yml
	  # 安装依赖
      - name: Install dependencies
        run: pnpm install

      # 部署
      - name: run deploy.sh # 步骤3
        env: # 设置环境变量
          GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }} # toKen私密变量
        run: pnpm run deploy
```

## 部署脚本

上面的工作流中运行了`pnpm run deploy`，这个其实是运行了一个sh脚本：

```json
 "scripts": {
    "build": "tsc && vite build",
    "build-github": "tsc && vite build --mode github",
    "deploy": "sh scripts/deploy.sh",
  },
```

脚本内容：

```sh
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run build-github

# 进入生成的文件夹
cd dist

# deploy to github pages
# 检查环境变量 GITHUB_TOKEN 是否为空。如果 GITHUB_TOKEN 变量为空，那么条件语句中的代码块将被执行。
if [ -z "$GITHUB_TOKEN" ]; then
  msg='deploy'
  githubUrl=git@github.com:Michael-py001/react-slide.git
else
  # 这是github actions执行的代码块
  msg='来自github actions的自动部署'
  githubUrl=https://Michael-py001:${GITHUB_TOKEN}@github.com/Michael-py001/react-slide.git
  git config --global user.name "Michael"
  git config --global user.email "969149642@qq.com"
fi
git init
git add -A
git commit -m "${msg}"
git push -f $githubUrl master:gh-pages # 推送到github gh-pages分支
```

这个脚本内容大概就是：

1. 运行`npm run build-github`，打包项目
2. 查看是否有GITHUB_TOKEN环境变量，设置不同的githubUrl，供后面push使用
3. 进入dist后初始化仓库 ，强制提交内容到githubUrl地址

构建产物提交到gh-pages后，只需要开启github page就能实现自动部署了，这个分支只要有新的提交，github就会自动触发一个action，会帮你把分支上的旧产物删除，只保留新提交的产物，所以就不用自己删除dist里的文件了。

并且只保留最新一条提交记录。



![image-20230928174551551](https://s2.loli.net/2023/09/28/sEhNaoURA1bgMIu.png)

### 开启github page

![image-20230928181504224](https://s2.loli.net/2023/09/28/Pf7ETuijr1K92Do.png)





这里用到的`GITHUB_TOKEN`是在工作流中设置的：

```yml
env: # 设置环境变量
   GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }} # toKen私密变量
```

这里用到的ACCESS_TOKEN，下面来介绍。

## 生成ACCESS_TOKEN访问令牌

需要在个人GitHub账号中进行创建：

`Settings`---> `Developer settings`--->`Personal access tokens`---> `Tokens(classic)`。

权限默认即可。

![image-20230928163808085](https://s2.loli.net/2023/09/28/6g7LY85KFPJthdE.png)

生成令牌后，去仓库的设置中用一个环境变量(ACCESS_TOKEN)存起来。

![image-20230928164106009](https://s2.loli.net/2023/09/28/hS8icOzPBZDV54E.png)

## 关于推送后不会触发github action的问题

> [推送事件不会触发推送路径上的工作流（github操作） - 我爱学习网 (5axxw.com)](https://www.5axxw.com/questions/content/tufs4h)

你只需要一个不同的令牌。

当您使用actions/checkout时，它使用`GITHUB_TOKEN`进行身份验证，并且根据文档，它不会触发新的工作流运行：

> 当您使用存储库的GITHUB_TOKEN代表GitHub Actions应用程序执行任务时，GITHUB_TOKEN触发的事件将不会创建新的工作流运行。这样可以防止意外创建递归工作流运行。

要使其工作，您需要生成PAT（个人访问令牌），将其存储在存储库机密中，并在签出步骤中使用它：

```yml
- uses: actions/checkout@v2
  with:
    token: ${{ secrets.YOUR_PAT_TOKEN }}
```

## 参考文章

> [GitHub配置自动部署pages与服务器 | 二丫讲梵 (eryajf.net)](https://wiki.eryajf.net/pages/47a507/)
# 20241129-Github Action 实现自动部署github-page

## 配置script命令

![image-20241129182132117](https://s2.loli.net/2024/11/29/pWLne3BjGXTHOx2.png)

`build-github`命令使用了指定的mode，是专门为了构建到github-page上的，对一些静态资源文件做了路径处理：

![image-20241129182745099](https://s2.loli.net/2024/11/29/7RWl4fUV9zCnAJ5.png)

![image-20241129182711117](https://s2.loli.net/2024/11/29/XISupiYRfZHQenx.png)

## sh部署脚本

```sh
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run build-github

# 进入生成的文件夹
cd dist

# 清理旧的 git 信息（如果存在）
rm -rf .git

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

# 创建新的git仓库
git init

# 切换到新的分支（不使用gh-pages作为分支名）
git checkout -b temp-deploy

git add -A
git commit -m "${msg}"

# 强制推送到远程gh-pages分支，使用temp-deploy:gh-pages避免分支名冲突
git push -f $githubUrl temp-deploy:gh-pages

# 清理
cd -
rm -rf dist
```

> ## 脚本解析
>
> -z 是一个测试操作符，用于检查字符串是否为空（zero length）
>
> $GITHUB_TOKEN 是一个环境变量
>
> 如果没有设置 GITHUB_TOKEN（通常是在本地开发环境）：
>
> 使用 SSH 方式的 URL（git@github.com:...）
>
> 提交信息设置为简单的 'deploy'
>
> 如果设置了 GITHUB_TOKEN（通常是在 GitHub Actions 环境中）：
>
> 使用 HTTPS 方式的 URL，并包含 token 进行认证
>
> 设置 Git 提交者信息为 GitHub Actions 机器人
>
> 提交信息设置为 '来自github actions的自动部署'
>
> 常用的 shell 测试操作符：
>
> -z : 检查字符串是否为空
>
> -n : 检查字符串是否非空
>
> -d : 检查目录是否存在
>
> -f : 检查文件是否存在
>
> -e : 检查文件或目录是否存在

## workflows工作流脚本

```sh
name: Build and Deploy
on:
  push:
    branches:
      - master
  #pull request时也触发
  pull_request:
    branches:
      - master
permissions:
  contents: write
jobs:
  build-and-deploy:
    concurrency: ci-${{ github.ref }} # Recommended if you intend to make multiple deployments in quick succession.
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x]
    steps:
      # 检出代码
      - name: Checkout 🛎️
        uses: actions/checkout@v3

      # 安装nodejs
      - name: Use Node.js ${{ matrix.node-version }} # 步骤2
        uses: actions/setup-node@v3 # 作用：安装nodejs
        with:
          node-version: ${{ matrix.node-version }} # 版本

      # 安装pnpm
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        id: pnpm-install
        with:
          version: 8.7.0
          run_install: false

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

      # 安装依赖
      - name: Install dependencies
        run: pnpm install

      # 部署
      - name: run deploy.sh # 步骤3
        env: # 设置环境变量
          GITHUB_TOKEN: ${{ secrets.ACESS_TOKEN }} # toKen私密变量
        run: pnpm run deploy

```


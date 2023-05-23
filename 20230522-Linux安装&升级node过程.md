# 20230522-Linux安装&升级node过程

> node下载地址
> [Index of /dist/latest-v16.x/ (nodejs.org)](https://nodejs.org/dist/latest-v16.x/)

## 使用nvm安装

> nvm为了方便安装多个版本的node

1. 安装 nvm

   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
   ```

   

​	安装完成后，重新打开终端或执行以下命令使 nvm 生效：

```bash
source ~/.bashrc
```

2. 查看可用的Node.js版本

   ```bash
   nvm ls-remote
   ```

   这会列出所有可用的 Node.js 版本，你可以选择一个你想要安装的版本。

3. 安装新的 Node.js 版本

   在终端中执行以下命令安装一个新的 Node.js 版本：

   ```
   nvm install <version>
   ```

   其中 `<version>` 是你想要安装的 Node.js 版本号，例如：

   ```
   nvm install 16.13.0
   ```

   这会下载并安装 Node.js 16.13.0 版本

   

4. 切换到新的 Node.js 版本
   在终端中执行以下命令切换到一个已安装的 Node.js 版本：

   ```
   nvm use <version>
   ```

5. 升级到最新的 LTS 版本

​		如果你想升级到最新的 LTS 版本，可以执行以下命令：

```
nvm install --lts
```

这会下载并安装最新的 LTS 版本。

	6. 升级到最新的稳定版本	

```
nvm install node
nvm install node --reinstall-packages-from=node // 安装最新的 Node.js 版本，并将已安装的 npm 包重新安装到新的 Node.js 版本中。
```

需要注意的是，升级 Node.js 可能会导致一些依赖包不兼容，因此在升级之前，建议备份你的项目和依赖包，并进行充分的测试。

## 使用n工具升级node



1. 使用 npm 升级 Node.js

   ```bash
   npm install -g n
   sudo n stable
   ```

   

### n: 找不到命令

如果您在使用 `n` 命令时遇到了 `command not found` 或 `找不到命令` 的错误，可能是因为 `n` 工具没有被正确安装或配置。

以下是一些可能的解决方法：

1. 确认 `n` 工具已经正确安装

在终端中执行以下命令，确认 `n` 工具是否已经正确安装：

```
n --version
```

如果输出了 `n` 工具的版本号，则说明 `n` 工具已经正确安装。如果没有输出版本号，可能是因为 `n` 工具没有被正确安装或配置。

2. 确认 `n` 工具的安装路径

如果 `n` 工具已经正确安装，但是在终端中执行 `n` 命令时仍然出现了 `command not found` 或 `找不到命令` 的错误，可能是因为 `n` 工具的安装路径没有被正确配置。

您可以在终端中执行以下命令，查看 `n` 工具的安装路径：

```
which n
```

如果输出了 `n` 工具的安装路径，则说明 `n` 工具已经被正确配置。如果没有输出安装路径，可能是因为 `n` 工具的安装路径没有被正确配置。

3. 将 `n` 工具的安装路径添加到 PATH 环境变量中

如果 `n` 工具的安装路径没有被正确配置，您可以将 `n` 工具的安装路径添加到 PATH 环境变量中，以便在终端中执行 `n` 命令时能够找到 `n` 工具。

您可以在终端中执行以下命令，将 `n` 工具的安装路径添加到 PATH 环境变量中：

```bash
export PATH="$PATH:/path/to/n/bin"
```

其中 `/path/to/n/bin` 是 `n` 工具的安装路径。您需要将其替换为实际的安装路径。

如果您希望将 `n` 工具的安装路径永久添加到 PATH 环境变量中，可以将上面的命令添加到您的 shell 配置文件中，例如 `~/.bashrc` 或 `~/.zshrc`。

需要注意的是，不同的操作系统和终端可能有不同的配置方法，请根据您的实际情况进行调整。

## 用二进制包更新Node.js

要使用二进制包更新 Node.js，可以按照以下步骤进行操作：

1. 下载新的 Node.js 二进制包

在 Node.js 官网上下载您想要安装的 Node.js 版本的二进制包。下载地址为：[https://nodejs.org/en/download/](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)

请注意选择与您的操作系统和 CPU 架构相对应的二进制包。

或者使用以下命令下载:

```bash
weget https://nodejs.org/dist/latest-v16.x/node-v16.20.0-linux-x64.tar.xz 
```

2. 解压二进制包

将下载的二进制包解压到您想要安装 Node.js 的目录中。例如，如果您想要将 Node.js 安装到 `/usr/local/node` 目录中，可以执行以下命令：

```bash
sudo tar -xvf node-v14.18.1-linux-x64.tar.xz -C /usr/local/node --strip-components=1
```

上面的命令会将 `node-v14.18.1-linux-x64.tar.xz` 文件解压到 `/usr/local/node` 目录中，并将解压后的文件夹重命名为 `node`。

3. 配置环境变量

​	将 Node.js 的可执行文件路径添加到 PATH 环境变量中，以便在终端中可以直接执行 Node.js 命令。

您可以在终端中执行以下命令，将 Node.js 的可执行文件路径添加到 PATH 环境变量中：

```bash
export PATH="$PATH:/usr/local/node/bin"
```

如果您希望将 Node.js 的可执行文件路径永久添加到 PATH 环境变量中，可以将上面的命令添加到您的 shell 配置文件中，例如 `~/.bashrc` 或 `~/.zshrc`。

4. 验证安装

   ```bash
   node -v
   ```

   如果输出了 Node.js 的版本号，则说明安装成功。

   需要注意的是，使用二进制包安装 Node.js 可能会导致一些依赖包不兼容，因此在安装之前，建议备份您的项目和依赖包，并进行充分的测试。

### 命令解析

`sudo tar -xvf node-v14.18.1-linux-x64.tar.xz -C /usr/local/node --strip-components=1`

这个命令的作用是将 Node.js 的二进制包解压到 `/usr/local/node` 目录中，并将解压后的文件夹重命名为 `node`。

具体来说，这个命令包含以下参数：

- `sudo`：以管理员权限执行命令，因为在 `/usr/local` 目录下需要管理员权限才能写入文件。
- `tar`：解压工具，用于解压 Node.js 的二进制包。
- `-xvf`：`tar` 命令的参数，表示解压并显示详细信息。
- `node-v14.18.1-linux-x64.tar.xz`：要解压的 Node.js 二进制包的文件名。
- `-C /usr/local/node`：`tar` 命令的参数，表示将解压后的文件夹放到 `/usr/local/node` 目录中。
- `--strip-components=1`：`tar` 命令的参数，表示解压后的文件夹中去掉一级目录，即将 `node-v14.18.1-linux-x64` 文件夹重命名为 `node`。

因此，执行这个命令后，Node.js 的可执行文件将会被安装到 `/usr/local/node/bin` 目录中，您可以将这个目录添加到 PATH 环境变量中，以便在终端中可以直接执行 Node.js 命令。

## 报错：Error: Cannot find module 'node:path'

安装Node到16.x.x版本即可解决

## 使用ES-Module语法

v13.2.0版本后，可以直接使用ES modules

### 在node中使用ES modules

1. 文件后缀名使用`.mjs`

2. 文件依然使用`.js`的后缀名，在项目的`package.json`中设置：`type:module`
   


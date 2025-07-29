# 20250729-【Git】Git config 配置

## git pull 合并配置

执行 git pull 时，如果出现以下提示：

```
fatal: Need to specify how to reconcile divergent branches.
hint: You have divergent branches and need to specify how to reconcile them.
...
```

原因：

- 你的本地分支和远程分支有“分叉”（divergent），即两边都有各自的提交，无法直接合并。

- 新版 Git（2.27+）在遇到这种情况时，不再默认用 merge 或 rebase，需要你明确指定合并策略。

解决方法：

你可以任选以下三种方式之一：

1. 合并（merge，保留分叉历史）

   ```bash
      git pull --no-rebase
   ```

或设置为默认：

```
   git config pull.rebase false
```

2. 变基（rebase，线性历史）

   ```bash
      git pull --rebase
   ```

​	或设置为默认：

```bash
   git config pull.rebase true
```

3. 只允许快进（fast-forward only，不允许分叉）

   ```bash
      git pull --ff-only
   ```

   或设置为默认：

   ```bash
      git config pull.ff only
   ```

推荐做法：

- 如果你不确定，建议用 merge（最安全）：

  ```bash
    git pull --no-rebase
  ```

- 如果你喜欢线性历史，可以用 rebase：

  ```bash
    git pull --rebase
  ```

全局设置（所有仓库生效）：

```bash
git config --global pull.rebase false   # 默认merge
# 或
git config --global pull.rebase true    # 默认rebase
```

### 解释

#### 1. git pull --no-rebase

- 含义：拉取远程分支时，使用“合并（merge）”的方式，把远程的新提交和你本地的提交合并在一起。

- 效果：如果两边都有新提交，会产生一个“合并提交（merge commit）”，保留分叉历史。

- 适用场景：适合团队协作，保留所有开发分支的历史脉络。

------

#### 2. git config pull.rebase false

- 含义：设置 Git 的默认拉取策略为“合并（merge）”，等价于每次 git pull 都加 --no-rebase。

- 用法：可以加 --global 让所有仓库都生效。

- 效果：以后直接 git pull，默认就是合并，不用每次都加参数。

------

#### 3. git pull --rebase

- 含义：拉取远程分支时，使用“变基（rebase）”的方式，把你本地的提交“挪到”远程最新提交之后。

- 效果：不会产生合并提交，历史记录更线性、整洁。

- 适用场景：适合个人开发或追求简洁历史的团队，但有冲突时需要手动解决。

------

#### 4. git pull --ff-only

- 含义：只允许“快进合并（fast-forward）”，即本地分支必须是远程分支的直接子集。

- 效果：如果本地有自己的提交，或者远程有新的提交，不能直接快进，就会报错，不会自动合并或变基。

- 适用场景：适合对历史极度敏感、只允许线性提交的场景（如主分支保护）。



#### 总结

- --no-rebase/pull.rebase false：合并，保留分叉历史。

- --rebase/pull.rebase true：变基，历史线性。

- --ff-only：只允许快进，不允许分叉或合并。

## git 凭证缓存

每次 git push 都要输入密码，常见原因如下：

1. 使用 HTTPS 协议

你的远程仓库地址是 https://...，每次推送都需要输入用户名和密码（或 token）。

2. 未配置凭证缓存

没有设置 Git 的凭证管理器（credential helper），Git 不会记住你的密码。

3. 未使用 SSH 密钥

如果用 SSH 协议（git@...），配置好密钥后就不需要每次输入密码。

**解决方法：**

### 方案一：配置 SSH 密钥（推荐，安全且免输密码）

1. 生成 SSH 密钥（如还没有）

   ```bash
      ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   按提示操作即可。

2. 添加公钥到 Git 服务器（如 GitHub/GitLab）

   - 查看公钥内容：

     ```bash
          cat ~/.ssh/id_ed25519.pub
     ```

   - 复制内容，添加到你的 Git 账户的 SSH Keys 设置里。

3. 修改远程仓库地址为 SSH

   ```bash
      git remote set-url origin git@github.com:yourname/yourrepo.git方案二：配置 HTTPS 凭证缓存
   ```

### 方案二：配置 HTTPS 凭证缓存

1. 开启凭证缓存（仅当前会话，适合临时用）

   ```bash
     git config --global credential.helper cache
   ```

2. 永久保存（推荐）

   ```bash
     git config --global credential.helper store
   ```

第一次输入密码后，Git 会将其保存在本地纯文本文件中（安全性较低）。

总结：

- 推荐用 SSH 免密推送，安全且方便。

- HTTPS 可用凭证管理器自动记住密码。
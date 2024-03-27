# 20240123-【Git】使用git stash暂存工作区的修改

`git stash` 是一个非常有用的命令，它可以帮助你保存当前的工作进度，让你可以切换到其他分支去做其他的工作，然后再回来继续你的工作。

## 常用命令

以下是一些常用的 `git stash` 命令：

1. `git stash save "your message"`：将当前的工作进度保存到一个新的 stash 中，你可以提供一个消息来描述这个 stash。
2. `git stash list`：列出所有的 stash。
   ![image-20240123114711197](https://s2.loli.net/2024/01/23/zGb2e4p7kMjBIQh.png)
3. `git stash apply stash@{n}`：应用指定的 stash，`n` 是你在 `git stash list` 中看到的 stash 的编号。这个命令会将 stash 的改动应用到当前的工作目录，但不会删除这个 stash。
4. `git stash pop stash@{n}`：应用指定的 stash，并删除这个 stash。
5. `git stash drop stash@{n}`：删除指定的 stash。
6. `git stash clear`：删除所有的 stash。

## 应用场景

使用 `git stash` 的一个典型的工作流程是：

1. 你正在一个分支上工作，但是需要切换到其他分支去做一些事情。
2. 你使用 `git stash save "your message"` 来保存你的工作进度。
3. 你使用 `git checkout` 切换到其他分支。
4. 你完成你的工作并提交你的改动。
5. 你使用 `git checkout` 切换回你原来的分支。
6. 你使用 `git stash pop` 来恢复你的工作进度。

这样，你就可以在不同的分支之间切换，而不会丢失你的工作进度。


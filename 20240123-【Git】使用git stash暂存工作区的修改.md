# 20240123-【Git】使用git stash暂存工作区的修改

在 Git 中，你可以使用 `git stash` 命令来暂存当前工作区的修改。这个命令会保存你的修改，并将工作区恢复到上次提交的状态。以下是一个示例：

```bash
git stash
```

在执行 `git stash` 命令后，你的修改会被保存在一个新的储藏中，你可以使用 `git stash list` 命令来查看所有的储藏：

```bash
git stash list
```

![image-20240123114711197](https://s2.loli.net/2024/01/23/zGb2e4p7kMjBIQh.png)

当你处理完其他事情后，你可以使用 `git stash apply` 命令来恢复你的修改。这个命令需要一个参数，这个参数是你想要恢复的储藏的名称。以下是一个示例：

```bash
git stash apply stash@{0}
```

在这个例子中，`stash@{0}` 是你想要恢复的储藏的名称。

请注意，`git stash apply` 命令只会恢复你的修改，但不会删除储藏。如果你想在恢复修改后删除储藏，你可以使用 `git stash pop` 命令：

```bash
git stash pop stash@{0}
```

在这个例子中，`stash@{0}` 是你想要恢复并删除的储藏的名称。
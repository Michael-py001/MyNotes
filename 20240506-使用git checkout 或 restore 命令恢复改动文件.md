# 20240506-使用git checkout 或 restore 命令恢复改动文件

## git checkout命令介绍

`git checkout` 是一个强大的命令，主要用于切换分支或恢复工作树文件。以下是一些常见的使用方式：

1. 切换到已存在的分支：`git checkout <branch_name>`
2. 创建新的分支并切换到该分支：`git checkout -b <new_branch_name>`
3. 恢复工作区的指定文件到最近一次commit或add的状态：`git checkout -- <file>`
4. 恢复整个工作区到最近一次commit或add的状态：`git checkout .`
5. 切换到上一个分支：`git checkout -`
6. 切换到某个commit：`git checkout <commit_hash>` 
7. `git checkout <branch> <file>` 从指定的 `<branch>` 分支获取 `<file>` 的版本，并将其复制到你当前的工作目录

注意：在使用 `git checkout <commit_hash>` 时，你会进入一个"HEAD detached"的状态，这意味着你不在任何分支上进行工作。在这种状态下的任何更改如果没有被保存在新的分支上，都会在切换到其他分支时丢失。

在Git 2.23.0版本后，`git checkout` 的部分功能已经被 `git switch` 和 `git restore` 命令替代，以使命令的功能更加明确。例如，切换分支应使用 `git switch <branch_name>`，恢复文件应使用 `git restore <file>`。

### 使用 git checkout 恢复其他分支的文件到当前分支

场景：当前在uat分支上，执行了`git checkout test packages\x-widgets packages\x-components` ，目的是想从 `test` 分支检出 `packages\x-widgets` 和 `packages\x-components` 这两个目录或文件到你当前的 `uat` 分支。

```sh
git checkout test packages\x-widgets\src\antdv\TimeRangePicker
```

## 使用 `git restore` 来替代 `git checkout <branch> <file>` 命令

你可以使用 `git restore` 来替代 `git checkout <branch> <file>` 命令。以下是如何使用 `git restore` 来实现相同的效果：

使用 `git restore` 从指定分支恢复文件

假设你有一个分支 `feature-branch`，并且你想从该分支恢复文件 [`index.vue`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)，你可以使用以下命令：

```bash
git restore --source feature-branch --staged --worktree index.vue
```

### 解释

- `--source <branch>`：指定从哪个分支恢复文件。
- `--staged`：将文件恢复到暂存区。
- `--worktree`：将文件恢复到工作区。

### 示例

假设你在当前分支上，并且想从 `feature-branch` 分支恢复 `index.vue` 文件：

```bash
git restore --source feature-branch --staged --worktree index.vue
```

这将从 `feature-branch` 分支恢复 [`index.vue`](vscode-file://vscode-app/c:/Users/WuShiLi/AppData/Local/Programs/Microsoft VS Code Insiders/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html) 文件到当前分支的暂存区和工作区。

### 使用 `git switch` 切换分支

如果你只是想切换到另一个分支，可以使用 `git switch` 命令：

```bash
git switch feature-branch
```

### 总结

- 使用 `git restore --source <branch> --staged --worktree <file>` 来从指定分支恢复文件。
- 使用 `git switch <branch>` 来切换分支。

这样，你就可以使用 `git restore` 和 `git switch` 来替代 `git checkout <branch> <file>` 的功能。

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


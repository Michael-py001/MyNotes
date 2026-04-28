# 20260423-【Git】如何修改最近一次commit描述

要修改上一次提交的描述（commit message），可以使用 `git commit --amend` 命令。具体操作如下：

### 1. 修改最近一次提交的信息
```bash
git commit --amend -m "新的提交描述"
```
- `-m` 后面直接写新的提交信息。
- 如果不使用 `-m`，Git 会打开默认文本编辑器（如 Vim）让你编辑提交信息。

### 2. 如果只想进入编辑器修改（不使用 `-m`）
```bash
git commit --amend
```
然后编辑器中修改第一行的提交信息，保存退出即可。

### 注意事项
- **只适用于尚未推送到远程仓库的提交**。如果已经推送（push）过，修改历史后需要强制推送：`git push --force-with-lease`（更安全）或 `git push -f`。
- 如果上一次提交已经包含多个文件的更改，`--amend` 只会修改提交信息，不会改变文件内容。如果想同时追加新的修改，可以先 `git add` 文件，再执行 `git commit --amend`（这会合并到上一次提交中）。

### 示例
```bash
# 假设上次提交信息写错了
git commit --amend -m "fix: 修正登录按钮样式"
# 如果已经 push 到远程分支（且该分支只有你一个人使用）
git push --force-with-lease origin 你的分支名
```

这样就完成了对上一次提交描述的修改。
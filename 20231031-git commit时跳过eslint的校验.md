## 20231031-git commit时跳过eslint的校验

在 Git 中，你可以使用 `--no-verify` 选项来跳过 pre-commit 钩子，从而跳过 ESLint 校验。具体来说，你可以运行以下命令：

```bash
git commit --no-verify -m "Your commit message"
```

在这个命令中，`--no-verify` 选项表示跳过 pre-commit 钩子，`-m "Your commit message"` 表示设置提交信息。

需要注意的是，跳过 ESLint 校验可能会导致代码质量问题，你应该在提交代码之前确保代码符合 ESLint 规则。如果你遇到 ESLint 校验错误，你应该修复这些错误，而不是跳过校验。
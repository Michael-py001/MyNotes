# 20250429-git alias 简化git命令

使用 git alias 可以把包含多个处理逻辑的代码简化成一个命令，非常好用。

## 配置文件

`"C:\Users\[用户]\.gitconfig"`文件，修改或增加alias:

```
[alias]
 	lg = 'xxx'
```

lg 即为简写命令名称，使用时输入: `git lg`

## 美化log日志

```bash
lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

使用： `git lg`

效果：

![image-20250429150536351](https://s2.loli.net/2025/04/29/Fx9zKnfBR85mt2v.png)

## 简化合并代码

```bash
ct = "!f() { \
        paths=( \
            'apps/finance-admin/src/assets' \
            'packages/x-utils' \
            'packages/x-widgets' \
        ); \
        total_files=0; \
        for path in \"${paths[@]}\"; do \
            result=$(git checkout test \"$path\" 2>&1); \
            if [[ $result != *\"Already up to date\"* ]]; then \
                files_count=$(echo \"$result\" | grep -o 'Updated.*path' | grep -o '[0-9]\\+'); \
                total_files=$((total_files + files_count)); \
                echo \"Merged: $path (Changed files: $files_count)\"; \
            fi; \
        done; \
        if [ $# -gt 0 ]; then \
            for arg in \"$@\"; do \
                result=$(git checkout test \"$arg\" 2>&1); \
                if [[ $result != *\"Already up to date\"* ]]; then \
                    files_count=$(echo \"$result\" | grep -o 'Updated.*path' | grep -o '[0-9]\\+'); \
                    total_files=$((total_files + files_count)); \
                    echo \"Merged: $arg (Changed files: $files_count)\"; \
                fi; \
            done; \
        fi; \
        echo \"Total changed files: $total_files\"; \
    }; f"
```

![image-20250429150611945](https://s2.loli.net/2025/04/29/OMjtKbyYA7cNuv8.png)

使用：
支持传入多个路径

```bash
git ct packages/xxx/src/views/xxx packages/xxx/src/views/xxx
```

## 跳过lint检查

```bash
# 跳过lint校验提交的简写
	skip = "!f() { git commit -m $1 --no-verify; }; f"
```

使用：

```bash
git skip "临时提交，修复紧急bug"
```


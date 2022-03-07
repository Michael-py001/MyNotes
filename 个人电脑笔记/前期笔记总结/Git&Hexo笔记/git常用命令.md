### Git 设置.gitignore无效时

```cmd
git rm -r --cached .

git add .
git commit -m 'update gitignore'
git push origin master
```


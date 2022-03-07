[TOC]



# 项目部署到github注意事项

## 更改master分支名为main

> 因为现在的github默认分支改成了main，所以如果想部署到main分支，则需要额外的操作

- 第一种就是到github上修改main的分支名称为master，这样你git的操作不用改
- 第二种就是原有的项目中的master想提交到main分支，接下来进行讲解操作



## 设置追踪源，关联仓库

```
git remote add origin http://xxx.git（仓库地址）
```



## 2. 更改分支名称

> 如果本地的分支名为master

```
git branch -m 当前分支名 新分支名
git branch -m master main
```

接下来:

```
git add .
git commit -m "xxx"
git push -u origin main
```

如果出错，可以强制push:

```
 git push -u origin main -f
```



## 强制push

```
 git push -u origin master -f
```



## git push -u 参数

**-u选项会**指定一个默认主机，这样后面就可以不加任何参数使用git push。





## 将项目推送到分支上 如 devCode分支

  创建分支

```
git branch devCode
```

  切换分支

```Go
git checkout devCode
```

把分支添加到github上

```
git push --set-upstream origin devCode
```

## 删除分支

```
git push origin --delete master
```


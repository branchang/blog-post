---
title: "Git实用技巧记录"
date: 2019-01-20 21:30:00 +0800
categories: [Developer, Git]
tags: [Git]
---

## 更改远端路径

```bash
cd $REPOS
git remote set-url origin $NEW_REMOTE_URL
git remote set-url --push origin $NEW_REMOTE_URL
```

## 克隆项目子树

对于一个庞大的项目，很多时候我们只需要查看其中一小部分的代码，例如官方的实例代码，因此可以使用`spare checkout`功能达到这个目标：

```bash
mdkir $REPOS_NAME
cd $REPOS_NAME
git init
git remote add -f origin $REMOTE_REPOS_URL
```
开启`spare checkout`：

```bash
git config core.sparseCheckout true
```

此时检查`.git/config`会显示配置效果：

```bash
[core]
   ...
   sparseCheckout = true
   ...
```

新建文件`.git/info/sparse-checkout`，在此逐行声明需要的目标子目录即可，语法和`.git/ingnore`相似。

```bash
echo $TARGET_SUB_DIR >> .git/info/sparse-checkout
git pull origin master
```

## 参考

* [http://schacon.github.io/git/git-read-tree.html#_sparse_checkout](http://schacon.github.io/git/git-read-tree.html#_sparse_checkout)

* [https://stackoverflow.com/questions/600079/how-do-i-clone-a-subdirectory-only-of-a-git-repository](https://stackoverflow.com/questions/600079/how-do-i-clone-a-subdirectory-only-of-a-git-repository)

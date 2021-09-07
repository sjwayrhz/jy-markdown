# Git fork 别人代码后与其同步更新

[TOC]

## 同步更新前

### for 别人的仓库

现在以 gin-vue-admin 项目为例，它在github和gitee的地址如下

```
git@github.com:sjwayrhz/gin-vue-admin.git
git@gitee.com:sjwayrhz/gin-vue-admin.git
```

先fork作者的github仓库

得到自己的项目仓库地址为

```
git@github.com:sjwayrhz/gin-vue-admin.git
```

### 查看否建立了主repo的远程源

```bash
~]# git remote -v
origin  git@github.com:sjwayrhz/gin-vue-admin.git (fetch)
origin  git@github.com:sjwayrhz/gin-vue-admin.git (push)
```

### 添加主repo的源

```bash
~]# git remote add upstream git@github.com:flipped-aurora/gin-vue-admin.git
```

### 再检查主repo远程源

```bash
~]# git remote -v
origin  git@github.com:sjwayrhz/gin-vue-admin.git (fetch)
origin  git@github.com:sjwayrhz/gin-vue-admin.git (push)
upstream        git@github.com:flipped-aurora/gin-vue-admin.git (fetch)
upstream        git@github.com:flipped-aurora/gin-vue-admin.git (push)
```

## 同步更新时

### 拉取upstream更新

```bash
~]# git fetch upstream
```

这一步会把原仓库的master分支变为你仓库的upstream/master分支，其他分支变为对应的upstream/branchName，像下面这样

```
remote: Enumerating objects: 135, done.
remote: Counting objects: 100% (94/94), done.
remote: Compressing objects: 100% (31/31), done.
remote: Total 59 (delta 43), reused 36 (delta 26), pack-reused 0
Unpacking objects: 100% (59/59), 7.70 KiB | 4.00 KiB/s, done.
From github.com:flipped-aurora/gin-vue-admin
 * [new branch]      develop        -> upstream/develop
 * [new branch]      gva-vue3       -> upstream/gva-vue3
 * [new branch]      gva_gormv2_dev -> upstream/gva_gormv2_dev
 * [new branch]      gva_vue2       -> upstream/gva_vue2
 * [new branch]      gva_workflow   -> upstream/gva_workflow
 * [new branch]      master         -> upstream/master
```

### 合并

```bash
~]# git merge upstream/master
```

### 其他命令

```bash
 ~]# git branch -vv
* master   dec2440 [origin/master] Update README.md
  upstream dec2440 Update README.md
 ~]# git remote show origin

 ~]# git remote show upstream
```




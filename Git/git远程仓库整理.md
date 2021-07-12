

查看远程仓库

```
git branch -r

例：本次执行查询出如下远程分支
  origin/HEAD -> origin/master
  origin/dev01
  origin/dev02
  origin/dev03
  origin/master
  origin/stage003
  origin/stage01
  origin/stage02
  origin/stage03
  origin/stage04
```

删除远程分支

```
git push origin --delete $branch_name

例：删除远程的dev01分支
git push origin --delete dev01
To codehub-cn-east-2.devcloud.huaweicloud.com:ywz00001/cicd.git
 - [deleted]         dev01
```

删除本地分支

```
git branch

例：本次输出如下
  dev01
  dev02
  dev03
* master
  stage003
  stage01
  stage02
  stage03
  stage04
  
删除dev01
git branch -D dev01
```


使用git pull时，报错：The following untracked working tree files would be overwritten by merge

解决方法：

```
$ git fetch origin
$ git clean -f
$ git reset --hard origin/master
```


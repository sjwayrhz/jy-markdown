使用git pull时，报错：The following untracked working tree files would be overwritten by merge

解决方法：

```
$ git fetch origin
$ git clean -f
$ git reset --hard origin/master
```

使用git pull时,报错：fatal: refusing to merge unrelated histories
```
PS D:\codeup\markdown> git pull
fatal: refusing to merge unrelated histories
```
原因是两个分支是两个不同的版本，具有不同的提交历史
解决方法如下
```
PS D:\codeup\markdown> git pull origin develop --allow-unrelated-histories
From codeup.aliyun.com:60e566f6f00dfa2fa4cabdd6/important/markdown
 * branch            develop    -> FETCH_HEAD
Merge made by the 'recursive' strategy.
```
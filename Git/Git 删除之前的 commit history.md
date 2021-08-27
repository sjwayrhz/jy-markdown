在stackoverflow上面查到的清楚之前混乱commit history的方案：

1. 检出master

```text
git checkout --orphan ddmichael_branch
```

2. 暂存全部文件

```text
git add -A
```

3. 提交刚刚暂存的所有文件

```text
git commit -am "commit message"
```

4. 删除主线

```text
git branch -D master
```

5. 将目前这个ddmichael_branch重命名为master主线

```text
git branch -m master
```

6. 将更新后名称的主线强制推到remote repository里面

```text
git push -f origin master
```




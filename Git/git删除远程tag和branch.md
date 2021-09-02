删除远程Tag

显示本地 tag

```
git tag 
Remote_Systems_Operation
```
  

删除本地tag
```
git tag -d Remote_Systems_Operation 
```


用push, 删除远程tag

```
git push origin :refs/tags/Remote_Systems_Operation
```

删除远程分支
```
git branch -r -d origin/branch-name
git push origin :branch-name
```
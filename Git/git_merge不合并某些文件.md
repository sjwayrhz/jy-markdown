# git_merge不合并某些文件
创建我们的自定义merge driver：
```bash
$ git config --global merge.ours.driver true
```
在要被merge的分支上创建.gitattributes 文件，并且在文件中置顶不merge的文件名：
```bash
$ echo 'email.json merge=ours' >> .gitattributes
$ git add .gitattributes
$ git commit -m 'chore: Preserve email.json during merges'
```
回到要合并到的分支（注意形容词）执行merge：
```bash
(newbranch) $ git checkout master
(master) $ git merge newbranch
Auto-merging ...
Merge made by the 'recursive' strategy.
 demo-shared | 1 +
 1 file changed, 1 insertion(+)
```
经过以上步骤，我们指定的email.json就不会被合并咯 

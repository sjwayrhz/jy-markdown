
# git同时支持多个远程仓库

# Git 一个项目同时使用多个远程仓库

有时我们为了方便管理，会需要将同一个项目同时发布到多个远程仓库中。如下是实现方法：

## 多个仓库都需要push和pull

1.  在指定git服务器上创建一个新的git仓库
    
2.  执行  
`git remote add XXXX git@gitee.com:xxx/demo.git`
    
4.  合并节点(新代码库要求是新的)  
    `git pull XXXX master --allow-unrelated-histories`
    
5.  将本地分支推到新仓库  
    `git push XXXX master`
    
6.  分别操作两个仓库  
    `git pull origin master`  
    `git push origin master`  
    `git pull XXXX master`  
    `git push XXXX master`
    

以上操作就可以在想要推送时，手动推送到指定仓库。

## 新仓库只需要push, 不需要pull

如果第二个仓库只需要push，不需要pull,那可以做以下操作。  
这样的好处是一次推出就可以同时同步多个仓库  
`git remote set-url --add origin git@gitee.com:xxx/demo.git`

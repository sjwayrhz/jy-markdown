# Git多仓库同步

[TOC]



## 准备

需求： 自己在github上和gitee上都有仓库，gitee速度快，但是github更正式。

现在需要在本地开发的时候，全部使用的是gitee仓库的内容，然后在jenkins的服务器上创建定时任务，及时拉取最新的更新内容并推送到github。

### 本地场景

在gitee拉取代码之后，不对 .git/config 文件做修改。

### jenkins服务器场景

拉取代码

```bash
~]# cd /usr/local/src
src]# git clone git@gitee.com:sjwayrhz/devops.git
```

在git拉取代码之后，修改 .git/config文件，以devops仓库为例，配置文件如下：

```
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = git@gitee.com:sjwayrhz/devops.git
	fetch = +refs/heads/*:refs/remotes/origin/*
	url = git@github.com:sjwayrhz/devops.git
[branch "master"]
	remote = origin
	merge = refs/heads/master
[branch "develop"]
	remote = origin
	merge = refs/heads/develop
```

新建jenkins job，类型为pipeline，测试一个syncGit脚本，使其生成workspace.

检查workspace的地址为`/opt/jenkins/workspace/syncGit`,然后创建devops和syncGit之间的软连接

```bash
~]# ln -s /usr/local/src/devops /opt/jenkins/workspace/syncGit
```

编写jenkins脚本

```
```




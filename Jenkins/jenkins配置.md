## Jenkins相关配置

[TOC]

### 修改默认安装目录

打开tomcat的bin目录，编辑catalina.sh文件。

```
在# OS specific support.  $var _must_ be set to either true or false.上面添加：export JENKINS_HOME=""
```

在引号中填入你的路径。

### 设置语言为英文

安装插件“Locale”

jenkins->【系统管理】->【系统设置】->【Locale】，输入：

zh_CN	中文

en_US	英文

勾选`Ignore browser preference and force this language to all users`

可自行选择

### 安装插件

Git Parameter
Gitlab
GitHub Authentication
Publish Over SSH




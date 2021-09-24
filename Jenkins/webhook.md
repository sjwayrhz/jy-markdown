## Jenkins-Gitlab-Webhook

[TOC]

### 环境准备

项目的url为`git@gitlab.juneyaokc.com:caobo/helloworld.git`，新建pipeline项目为helloworld

Jenkins 添加GitLab Hook Plugin

### 在jenkins的helloworld项目中获取webhook的url

登录jenkins

```
Jenkins -> helloworld -> configure -> Build Triggers -> 

Build when a change is pushed to GitLab. GitLab webhook URL: http://192.168.177.57:8080/jenkins/project/helloworld
```

### 在gitlab中导入webhook

登录gitlab

```
Settings -> Intergrations
```

设置的值为

```
URL -> http://192.168.177.57:8080/jenkins/project/helloworld
```

其他值默认即可

### 关闭jenkins中的gitlab 验证(不安全)

```
Manage Jenkins -> Configure System -> Gitlab
```

取消 `Enable authentication for '/project' end-point`

### 生成access-token

如果关闭jenkins中的gitlab 验证，任何人只要知道你项目的钩子地址(webhook URL)就可以疯狂触发任务，所以建议在gitlab中生成access_token

```
Settings -> Access Tokens


 Add a personal access token

Pick a name for the application, and we'll give you a unique personal access token.
Name : jenkins-gitlab-webhook
Expires at : 2022-06-10
Scopes
api Access the authenticated user's API (√)
Full access to GitLab as the user, including read/write on all their groups and projects
read_user Read the authenticated user's personal information (×)
Read-only access to the user's profile information, like username, public email and full name
```

点击`create personal access token`

如此获得了access_token  --> `4sDPzVMpXurnsEixgdxb`

```
Your New Personal Access Token 

4sDPzVMpXurnsEixgdxb

Make sure you save it - you won't be able to access it again.
```

登录jenkins创建Credentials    

```
Manage Jenkins -> Manage Credentials -> Jenkins -> Global credentials (unrestricted) -> Add Credentials    
```

内容如下

```
Kind -> Gitlab API token
Scope -> Global (Jenkins,nodes,items,all child items,etc)
API token -> 4sDPzVMpXurnsEixgdxb
ID -> gitlab-access-token
Description -> null
```

配置jenkins中的gitlab 验证

```
Manage Jenkins -> Configure System -> Gitlab
```

勾选`Enable authentication for '/project' end-point`

```
GitLab connections
Connection name -> jy-gitlab
Gitlab host URL -> http://192.168.177.98:9999
Credentials -> Gitlab API token
```

点击`Test Connection` 测试是否连接成功

此时，gitlab中的webhook的url就不可以是匿名的URL了，需要使用jenkins的用户与tokens了。

```
语法为	->	   https://username:password@JENKINS_URL/project/PROJECT_NAME
举例	 ->	    http://caobo:1190b3b2ae7236f32cca63444aabdbc030@192.168.177.57:8080/jenkins/project/helloworld
```


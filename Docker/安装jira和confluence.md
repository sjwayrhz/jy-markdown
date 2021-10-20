## 说明
### 素材

本文采用素材如下：

Docker镜像 Github链接 `https://github.com/cptactionhank/`

破解工具 `git@gitee.com:pengzhile/atlassian-agent.git`

采用以上工具，理论上可以破解几乎全部版本。
### 数据库

如果是选择外部数据库，大家可以按照这样创建：
```
# 创建jira数据库及用户
CREATE DATABASE jiradb CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on jiradb.* to 'jirauser'@'%' identified by 'tVxxxb6n';

# 创建confluence数据库及用户
CREATE DATABASE confdb CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on confdb.* to 'confuser'@'%' identified by '7FxxxzhO';

# confluence要求设置事务级别为READ-COMMITTED
set global tx_isolation='READ-COMMITTED';
```
## 安装jira

### 制作Docker破解容器
编写Dockerfile文件：
```
FROM cptactionhank/atlassian-jira-software:latest

USER root

COPY "atlassian-agent.jar" /opt/atlassian/jira/

RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
```
### 下载破解文件
在github 中下载编译好的即可，放置在Dockerfile同目录下
```
- JIRA
  --Dockerfile
  --atlassian-agent.jar
```

### 构建镜像
`docker build -t sjwayrhz/jira:latest .`
结果如下：
```
Sending build context to Docker daemon  2.141MB
Step 1/4 : FROM cptactionhank/atlassian-jira-software:latest
 ---> c51100467795
Step 2/4 : USER root
 ---> Running in 3f9cea0602c7
Removing intermediate container 3f9cea0602c7
 ---> 4b9e20ba43cf
Step 3/4 : COPY "atlassian-agent.jar" /opt/atlassian/jira/
 ---> 61155470b50a
Step 4/4 : RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
 ---> Running in 5aed1ac41ab7
Removing intermediate container 5aed1ac41ab7
 ---> 33d0b86f8262
Successfully built 33d0b86f8262
Successfully tagged sjwayrhz/jira:latest
```
### 启动容器
```
docker run -d --name jira \
  --restart always \
  -p 8090:8090 \
  -e TZ="Asia/Shanghai" \
  -m 4096M \
  v /home/data/www/jira.sjhz.tk:/var/atlassian/jira \
  sjwayrhz/jira:latest
```
访问 IP:8080，选择语言并选择手动配置,
演示使用内置数据库（生产环境需配置独立数据库）
设置属性
破解
```
java -jar atlassian-agent.jar -d -m i@sjhz.tk -n BAT -p jira -o http://jira.sjhz.tk -s BHGF-KF7K-WHP4-O23S

====================================================
=======     Atlassian Crack Agent v1.3.1     =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!):

AAAB5A0ODAoPeJyNU9mOozAQfOcrkPZxRMIlckiWJgGyYQOELGRW++iQzuAZAoxtcszXL1c0R6JoJ
b/Yququqm7/CDEXlzEXFUOU9bE2GKuaaIaRqMqqIjxTgCzJiwJozyUxZAyicwE+3gMyl55n/zadi
SuYFDAneWZhDqgmSoosKYZwh2IBiykpahZaZynZEw5bMW0J4uYsJpwXbNzvvyckhR7JBQ+TjEOGs
xjsU0Houes2HEnyoDrCC6H4otLekra07zqeE9mW4Jf7DdDlbs2AMiQpF3F3ahU035Yx79UXieU7f
sQUeleF7mBxzMkBEKclfMny8/sdeqUKm1C5pi20i+epalybU4Ww3HzE2EDsA07LZhhoh1PWlf9ea
EmfcUZYi6uTroJugOwlee/xV8HMM16JtKvQU0QeL88N5iqATtYcswR55tGcTbJ+kP+NjJGTzwPDD
U/lKvAWD29TazpJVpt++JTn9G11PNlRgHe/Hvx84evZejtBqG3xn/mEHNPaU+u0G6hjIdexQtuXX
MXQdE2TVV3Th6Mv+3FrJUOgB6AVfTr/OZMWs8FC+jMPdGmpaqHwCudL7IohywN5qGk3/8f15gUlj
RPM4Pvv+ExuZlNQwjrTlXx0w0I3lkb5dBL9A3E/Ro8wLAIUaGSbKRWeiMZcOJOwCoQGo4V0CSICF
E+88jgiNUFBJYlhdh7n9T+oF+ECX02mu
```
将生成的许可证复制到页面，完成破解。

## 安装 Confluence.
编写Dockerfile文件：
```
FROM cptactionhank/atlassian-confluence:latest

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/confluence/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```
### 下载破解文件
在github 中下载编译好的即可，放置在Dockerfile同目录下
```
- Confluence
  --Dockerfile
  --atlassian-agent.jar
```
### 构建镜像
`docker build -t sjwayrhz/confluence:latest .`
结果如下：
```
Sending build context to Docker daemon  976.9kB
Step 1/4 : FROM cptactionhank/atlassian-confluence:latest
 ---> 080599d8b2d7
Step 2/4 : USER root
 ---> Running in 016cda821c07
Removing intermediate container 016cda821c07
 ---> 6506aa1b43c1
Step 3/4 : COPY "atlassian-agent.jar" /opt/atlassian/confluence/
 ---> 27ab3f8f23cc
Step 4/4 : RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
 ---> Running in 68588c4f146c
Removing intermediate container 68588c4f146c
 ---> 45a74f5420da
Successfully built 45a74f5420da
Successfully tagged sjwayrhz/confluence:latest
```
### 启动容器
```
docker run -d --name confluence \
  --restart always \
  -p 8090:8090 \
  -e TZ="Asia/Shanghai" \
  -v /home/data/www/confluence.sjhz.tk:/var/atlassian/confluence \
  sjwayrhz/confluence:latest
```
### 访问 confluence
访问 IP:8090，参照JIRA的安装流程，进行操作。可在引导过程中，与之前安装的JIRA进行绑定关联。
获取应用的时候，有两个选择项，Confluence Questions和 Confluence Team Calendars这里选择第一个
授权码
在下面输入授权码，只需要输入Confluence的授权码，其他的可以稍后再添加。
### 破解
生成confluence许可命令参照如下：
```
java -jar atlassian-agent.jar -d -m i@sjhz.tk -n BAT -p conf -o http://confluence.sjhz.tk -s B88F-V1GL-HIQ8-UA4Y

====================================================
=======     Atlassian Crack Agent v1.3.1     =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!):

AAABnQ0ODAoPeJxtUV1vozAQfPevQLrHigRDD7hIlkqAfJwgSUPStI8OtyluiINsk5T++jOE6KRTJ
b/szHpndvZHRpWxzJWBXcNyRo47wo4RZhvDtmyMQgFUsTOPqALSIia2TOyi+ELLumPIgZYSUAQyF
6zqkC0v2Ykp+GOULAcuwdg3RqFUJUfD4VfBShiwM1qKd8qZvA1pWU3mZ34oa+A5DORH8TVQR9RCA
5ordgGiRA0oPHOl6zilrCTs6d7XS82oLEgaXsNJ+LF7dyarJB3WkYdXz28b/5B90qNbToNiHTyMf
zdivYv27jW6VPYMl9H0dffaZFdCbqKZokKB6BfsoOQmsmkqWNATkHCZpvE6nAcJ0na4Ak619/izY
qLpI/N/mZanH+r/ziOSzKMsXpgJdp1Hx3Z/eo7nYZSBuIDQ9Nj3J+YLnibmbP7sm9vg8e2mrifSE
HjrqUviCM0LCNnmh13L8izfcfBd53sTq1rkBZXw/z379O7jbJTV+38H7dQ6C4v6tAexPGyl7iSmd
h0vyDfL9EfqQhoHm78HYMflMCwCFEGC+TW7sWRwr2Q9WLHGEm4MQlRaAhQ3SJf/EFAfkU+qG8O/5
jeHXjeXww==X02k0
```

## 破解应用商城的markdown
如果下载的markdown收费应用程序，则在应用管理中找到该应用程序，并复制该应用程序的应用秘钥。
例如本次得到的密钥是org.swift.confluence.markdown，然后做如下破解操作：

```
java -jar atlassian-agent.jar -d -m i@sjhz.tk -n BAT -p org.swift.confluence.markdown -o http://confluence.sjhz.tk -s B88F-V1GL-HIQ8-UA4Y

====================================================
=======     Atlassian Crack Agent v1.3.1     =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!):

AAABqA0ODAoPeJyNUl1vozAQfPevQLpnwJA0IZEsNQFyiS4fbb6qy5tDl+ALGM42ofTX1yFEd6pO6
kn2g3fk2Zmd/fYCr8YqUoaLDewN3cEQO4a/2eq36yBfAFUs5wFVQK4V08Gmi9GcRcAlbOsCljQD4
q8Wi3Dtz0ZzFF5oWjafSExTCSgAGQlWNJUdT1nGlG6Z3hiMY20kShVyaNvvCUvBYjlaiRPlTN5Ir
qgGo5zHaQk8Akv+St4tdUa5OFmyYrGy/gIzKs6vecUt4ApEIZgEokQJyM+5opEKF5SlhD3+H8lGU
aFpWiet5imVCVn4lT8J/HifHGw5XUSHYG8/x+V2Vr2d1u74NErWxxdWd35nhx9KTeMHyinY6kkcy
klWEYK0Dq2QU90vfCuYqNsZewMT9/X5Qpm2wi6ttQ2IC4hZQMaeNzH3zve5OZ09e+Zu1P2JzlDvQ
cjrKJ0exn3sdToOWpbZEcQq3kmNEdO5J/pvKU+liBIq4fMatAO587toUx7/hH3TFi6Jvubc6XW6P
W/Q7zr4oXdPo1me8Wj7hVndl/pNoA3pB42q7OwwLQIUV3be02pea2Ofq4C51W32BR4cvtkCFQCXX
evCcPcuRn6jQh77rBPGfuYz5A==X02kg
```

## 乱码问题
在我们正常安装之后，中文可能会有乱码，我们修改一下连接字符串，在 confluence 的家目录下面，有一个配置文件confluence.cfg.xml，找到hibernate.connection.url，在数据库字符串后面加上如下字符，整体结果如下：
```
~]# cat /home/data/www/confluence.sjhz.tk/confluence.cfg.xml | grep jdbc:mysql
    <property name="hibernate.connection.url">jdbc:mysql://10.230.7.33:3306/confdb?useUnicode=true&amp;characterEncoding=utf8</property>
```

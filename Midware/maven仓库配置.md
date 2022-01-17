下载安装

```bash
~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.2/binaries/apache-maven-3.8.2-bin.tar.gz
~]# tar -zxvf apache-maven-3.8.2-bin.tar.gz -C /usr/local
```

settings.xml文件会在两个目录下存在：

1、Maven安装目录（全局）：%MAVEN_HOME%\conf\settings.xml

2、用户安装目录（用户）：${user.home}\.m2\settings.xml

第一个是全局配置，第二个是用户配置。当两者都存在，它们的内容将被合并，特定于用户的settings.xml文件占主导地位。

如果从头开始创建用户特定的配置，可以将全局的settings.xml复制到${user.home}\.m2目录下。

我的Maven安装目录：（%MAVEN_HOME%）/usr/local/apache-maven-3.8.2/

我的用户安装目录：（${user.home}）/root/m.2/

移动

```bash
~]# cp /usr/local/apache-maven-3.8.2/conf/settings.xml ~/.m2
```

修改 

```bash
~]# vim ~/.m2/settings.xml
```


阿里maven镜像配置

setting.xml截取

```
  </mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```



maven配置环境变量

```
export MAVEN_HOME=/usr/local/apache-maven-3.8.2
export PATH=$MAVEN_HOME/bin:$PATH 
```


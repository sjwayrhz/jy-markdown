# 使用demp-spring尝试jenkinsfile

[TOC]

## 环境

### 基础配置

项目在Linux中的目录 `/usr/local/src/project/spring-k8s-configmap-demo`

jenkins工作空间目录 `/opt/jenkins/workspace/demo-spring`

项目与工作空间简历软连接，设置目录名称为warehouse

```bash
~]# ln -s /usr/local/src/project/spring-k8s-configmap-demo /opt/jenkins/workspace/demo-spring/warehouse
```

### 配置maven

在 "Manage Jenkins" -> "Global Tool Configuration"当中配置maven

```
Name		-> 	m2
MAVEN_HOME	->	/usr/local/apache-maven-3.8.2
```

### 配置sharelibrary


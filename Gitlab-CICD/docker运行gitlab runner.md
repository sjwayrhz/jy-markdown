# docker运行gitlab runner

## 1. 参考

```
https://docs.gitlab.com/runner/install/docker.html
https://docs.gitlab.com/runner/register/index.html#docker
https://docs.gitlab.com/12.10/runner/register/
```



## 2. 说明

1. gitlab runner 本身不在 gitlab 中，需要另外安装
2. 为了让 gitlab 感知和调用，gitlab runner要向gitlab注册才能被使用
3. 一个 gitlab Runner 可以注册到多个 gitlab
4. 一个 gitlab 也有由多个注册的 gitlab runner
5. gitlab runner 也有 windows 版本
6. gitlab runner 最关键的配置文件夹是 /etc/gitlab-runner

## 3. 安装

安装gitlab-runner-01

```bash
$ docker run -d --name gitlab-runner-01 --restart always \
    -v /srv/gitlab-runner-01/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:latest
```

安装gitlab-runner-02

```bash
$ docker run -d --name gitlab-runner-02 --restart always \
    -v /srv/gitlab-runner-02/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:latest
```

注意对关键配置文件夹 `/etc/gitlab-runner-01`和`/etc/gitlab-runner-02`  进行容器卷持久化，下面的注册也会用到

## 4. 注册

使用一次性容器来便捷注册，关键是用了上面安装时的同一个容器卷 /srv/gitlab-runner-01/config
 下面的url和token要根据自己的gitlab来获取，见下方"如何获取url和token"

设置gitlab-runner-01

```bash
$ docker run --rm -v /srv/gitlab-runner-01/config:/etc/gitlab-runner gitlab/gitlab-runner register \
  --non-interactive \
  --executor "docker" \
  --docker-image alpine:latest \
  --url "https://gitlab.sjhz.tk" \
  --registration-token "xYvoAnF8ARWZTs-D9twb" \
  --description "gitlab-runner-01" \
  --tag-list "docker,localMachine" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

设置gitlab-runner-02

```bash
$ docker run --rm -v /srv/gitlab-runner-02/config:/etc/gitlab-runner gitlab/gitlab-runner register \
  --non-interactive \
  --executor "docker" \
  --docker-image alpine:latest \
  --url "https://gitlab.sjhz.tk" \
  --registration-token "xYvoAnF8ARWZTs-D9twb" \
  --description "gitlab-runner-02" \
  --tag-list "docker,localMachine" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

解释下上面的参数：

1. --executor指明runner使用的执行器。(执行器介绍见： [https://docs.gitlab.com/runner/executors/index.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.gitlab.com%2Frunner%2Fexecutors%2Findex.html))， 一般选择 docker
2. --docker-image 如果执行器选择了docker，则要选择一个默认的docker镜像，在.gitlab-ci.yml中没有指定image参数时采用。
3. --tag-list 是要为gitlab runner 设置的tag，可以为以后工程指定runner提供tag

如果注册成功，会在gitlab主页上的runners看到注册的runner信息

## 5. 如何获取url和token

## 6. 测试gitlab runner是否可用

在gitlab上添加一个 test 工程
 添加文件 .gitlab-ci.yml，写入如下内容

```yaml
image: gcc

build:
  stage: build
  script:
    - echo build

test:
  stage: test
  script:
    - echo test
```

提交后，去看CI/CD的执行情况，出现如下情况，标明 runner 可用：

## 7. gitlab-runner 注销

尝试如下注销指定 gitlab ci 的方式，注销失败

```
gitlab-runner unregister --url "http://10.168.1.108:8929" --token "aQqLxxoTVCdyin73x86t"
```

使用如下方式，注销成功

````
gitlab-runner unregister --all-runners
````


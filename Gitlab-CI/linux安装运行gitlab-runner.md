安装，略

安装成功后，检查版本如下：

```bash
$ gitlab-runner -v
Version:      14.6.0
Git revision: 5316d4ac
Git branch:   14-6-stable
GO version:   go1.13.8
Built:        2021-12-17T17:36:04+0000
OS/Arch:      linux/amd64
```

### 注册到gitlab

```bash
sudo gitlab-runner register -n \
  --url https://gitlab.sjhz.tk \
  --registration-token 7gxAzT7gR8SBBjFkz9bx \
  --executor docker \
  --description "runner linux" \
  --docker-image "docker:19.03.12" \
  --docker-privileged \
  --docker-volumes "/certs/client" \
  --tag-list "linux" 
```

检测生成的config.toml

```bash
$ cat /etc/gitlab-runner/config.toml
```

看到config.toml如下

```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "runner linux"
  url = "https://gitlab.sjhz.tk"
  token = "jUcwT2Jx55YQ8n6DukxW"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:19.03.12"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/certs/client", "/cache"]
    shm_size = 0
```

现在尝试dind service

```yaml
image: docker:19.03.12

variables:
  # When you use the dind service, you must instruct Docker to talk with
  # the daemon started inside of the service. The daemon is available
  # with a network connection instead of the default
  # /var/run/docker.sock socket. Docker 19.03 does this automatically
  # by setting the DOCKER_HOST in
  # https://github.com/docker-library/docker/blob/d45051476babc297257df490d22cbd806f1b11e4/19.03/docker-entrypoint.sh#L23-L29
  #
  # The 'docker' hostname is the alias of the service container as described at
  # https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#accessing-the-services.
  #
  # Specify to Docker where to create the certificates. Docker
  # creates them automatically on boot, and creates
  # `/certs/client` to share between the service and job
  # container, thanks to volume mount from config.toml
  DOCKER_TLS_CERTDIR: "/certs"

services:
  - docker:19.03.12-dind

before_script:
  - docker info

build:
  stage: build
  tags:
    - linux
  script:
    - docker build -t my-docker-image .
    - docker run my-docker-image /script/to/run/tests

```


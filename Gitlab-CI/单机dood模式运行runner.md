# 单机dood模式运行runner

To use Docker-in-Docker with TLS enabled:

pull docker image

```bash
$ docker pull gitlab/gitlab-runner:v14.6.0
```

Register GitLab Runner from the command line. Use `docker` and `privileged` mode:

```bash
$ docker run --rm -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:v14.6.0 register \
  --non-interactive \
  --url "https://gitlab.sjhz.tk" \
  --registration-token MCssk1xsjZo3yp4GyjAD \
  --executor docker \
  --description "dood" \
  --docker-image "docker:20.10.12" \
  --docker-volumes "/var/run/docker.sock:/var/run/docker.sock" \
  --tag-list "dood" 
```

- This command registers a new runner to use the `docker:19.03.12` image. To start the build and service containers, it uses the `privileged` mode. If you want to use [Docker-in-Docker](https://www.docker.com/blog/docker-can-now-run-within-docker/), you must always use `privileged = true` in your Docker containers.
- This command mounts `/certs/client` for the service and build container, which is needed for the Docker client to use the certificates in that directory. For more information on how Docker with TLS works, see https://hub.docker.com/_/docker/#tls.

The previous command creates a `config.toml` entry similar to this:

```toml
$ cat /srv/gitlab-runner/config/config.toml

concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "dood"
  url = "https://gitlab.sjhz.tk"
  token = "F1xKekgE5yD77rzn5fzg"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:20.10.12"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
```

Install [GitLab Runner](https://docs.gitlab.com/runner/install/).

```bash
$ docker run -d --name gitlab-runner --restart always \
    -v /srv/gitlab-runner/config:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:v14.6.0
```

You can now use `docker` in the job script. Note the inclusion of the `docker:19.03.12-dind` service:

```bash
image: docker:19.03.12

before_script:
  - docker info

build:
  stage: build
  tags:
    - dood
  script:
    - docker build -t my-docker-image:test .
    - docker run --rm my-docker-image:test
    - docker rmi -f my-docker-image:test
```


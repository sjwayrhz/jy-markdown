To use Docker-in-Docker with TLS enabled:

1. Install [GitLab Runner](https://docs.gitlab.com/runner/install/).

   ```bash
   $ docker run -d --name gitlab-runner-dind --restart always \
       -v /srv/gitlab-runner-dind/config:/etc/gitlab-runner \
       -v /var/run/docker.sock:/var/run/docker.sock \
       gitlab/gitlab-runner:latest
   ```

   Register GitLab Runner from the command line. Use `docker` and `privileged` mode:

   ```bash
   $ sudo gitlab-runner register -n \
     --url "https://gitlab.sjhz.tk" \
     --registration-token xYvoAnF8ARWZTs-D9twb \
     --executor docker \
     --description "runner dind" \
     --docker-image "docker:19.03.12" \
     --docker-privileged \
     --docker-volumes "/certs/client" \
     --tag-list "dind" 
   ```

- This command registers a new runner to use the `docker:19.03.12` image. To start the build and service containers, it uses the `privileged` mode. If you want to use [Docker-in-Docker](https://www.docker.com/blog/docker-can-now-run-within-docker/), you must always use `privileged = true` in your Docker containers.
- This command mounts `/certs/client` for the service and build container, which is needed for the Docker client to use the certificates in that directory. For more information on how Docker with TLS works, see https://hub.docker.com/_/docker/#tls.

The previous command creates a `config.toml` entry similar to this:

```toml
[[runners]]
  url = "https://gitlab.sjhz.tk"
  token = TOKEN
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "docker:19.03.12"
    privileged = true
    disable_cache = false
    volumes = ["/certs/client", "/cache"]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

You can now use `docker` in the job script. Note the inclusion of the `docker:19.03.12-dind` service:

```bash
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
  script:
    - docker build -t my-docker-image .
    - docker run my-docker-image /script/to/run/tests
```


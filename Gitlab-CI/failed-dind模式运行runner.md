来源：`https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#tls-enabled`

# dind模式运行runner

To use Docker-in-Docker with TLS enabled:

1. Install GitLab Runner. 

   ```bash
   $ docker pull gitlab/gitlab-runner:v14.6.0
   ```

   

2. Register GitLab Runner from the command line. Use `docker` and `privileged` mode:

   ```bash
   $ docker run --rm -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:v14.6.0 register \
     --non-interactive \
     --url "https://gitlab.sjhz.tk" \
     --registration-token ZAz-UHaBnU3z3uasx6vS \
     --executor docker \
     --description "runner dind" \
     --docker-image "docker:19.03.12" \
     --docker-privileged \
     --docker-volumes "/certs/client" \
     --tag-list "dind" 
   ```

   This command registers a new runner to use the `docker:19.03.12` image. To start the build and service containers, it uses the `privileged` mode. If you want to use [Docker-in-Docker](https://www.docker.com/blog/docker-can-now-run-within-docker/), you must always use `privileged = true` in your Docker containers.

   This command mounts `/certs/client` for the service and build container, which is needed for the Docker client to use the certificates in that directory. For more information on how Docker with TLS works, see https://hub.docker.com/_/docker/#tls.

   The previous command creates a `config.toml` entry similar to this:

   ```toml
   $ cat /srv/gitlab-runner/config/config.toml
   
   concurrent = 1
   check_interval = 0
   
   [session_server]
     session_timeout = 1800
   
   [[runners]]
     name = "runner dind"
     url = "https://gitlab.sjhz.tk"
     token = "1jG8xAghzz4dzHg4emye"
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

3. start gitlab-runner

   ```bash
    $ docker run -d --name gitlab-runner --restart always \
       -v /srv/gitlab-runner/config:/etc/gitlab-runner \
       -v /var/run/docker.sock:/var/run/docker.sock \
       gitlab/gitlab-runner:v14.6.0
   ```

   

4. You can now use `docker` in the job script. Note the inclusion of the `docker:19.03.12-dind` service:

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
       - dind
     script:
       - docker build -t my-docker-image .
       - docker run my-docker-image /script/to/run/tests
   ```
   
   


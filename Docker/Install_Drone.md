## 安装drone和gitlab

假设安装在 192.168.177.205 使用docker启动drone和drone-runner，代码如下：

启动drone
```bash
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITLAB_SERVER=http://192.168.177.211:30180 \
  --env=DRONE_GITLAB_CLIENT_ID=b5dfdf928082a9688382eb7e2e2c3c1a6e2b0a8d9cf2e3c0ff5db4860158e10d \
  --env=DRONE_GITLAB_CLIENT_SECRET=1adf90134fad891820265fbde2bc5432868d40a8631a36bcc54a2b0204734e72 \
  --env=DRONE_RPC_SECRET=34eee53878e75447940213d28c369b43 \
  --env=DRONE_SERVER_HOST=192.168.177.205 \
  --env=DRONE_SERVER_PROTO=http \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```
启动drone-runner
```bash
docker run --detach \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --env=DRONE_RPC_PROTO=http \
  --env=DRONE_RPC_HOST=192.168.177.205 \
  --env=DRONE_RPC_SECRET=34eee53878e75447940213d28c369b43 \
  --env=DRONE_RUNNER_CAPACITY=2 \
  --env=DRONE_RUNNER_NAME=my-first-runner \
  --publish=3000:3000 \
  --restart=always \
  --name=runner \
  drone/drone-runner-docker:1
```
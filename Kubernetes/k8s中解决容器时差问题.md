1、通过设置pod 模板中的环境变量 env解决

   在pod的模板中添加以下：
```
apiVersion: v1

kind: Pod
metadata:
  name: pod-name
spec:
  containers:
  - name: name
    image: image-name
    imagePullPolicy: IfNotPresent
    env:
    - name: TZ
      value: Asia/Shanghai
```
2、挂载宿主机的时区文件进行设置
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-tz
spec:
  containers:
  - name: ngx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: config
      mountPath: /etc/localtime
      readOnly: true
   volumes:
    - name: config
      hostPath:
        path: /etc/localtime
```
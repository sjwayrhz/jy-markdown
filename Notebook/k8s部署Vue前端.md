# [k8s部署Vue前端](https://www.cnblogs.com/sunju/articles/12882348.html)

我们用configMap形式来部署动态前端热更新
首先创建前端目录

```
mkdir /data/frontend
```

在目录里创建nginx虚拟机配置文件



```
cat frontend.conf
 server   {
           listen 80;
           server_name localhost;
           charset utf-8;
           index index.html index.htm index.jsp;

  location ~ {

        root /etc/nginx/micro_vue;

        }

}
```



创建

```
kubectl create configmap nginx-frontend --from-file=./frontend.conf
```

编写dockerfile,把前端dist打包好的文件放到前端项目目录里

```
FROM nginx:1.18.0
MAINTAINER sunju@logwsd.com
COPY dist /etc/nginx/micro_vue
CMD [ "nginx", "-g", "daemon off;"]
```

创建dockerfile,并且推送到私服仓库

```
docker build -t 172.16.0.12:6166/micro/frontend .
```

到这里前端应用就部署妥当了。
接下来用k8s 部署前端文件
部署前端yaml 文件，文件中添加了标签

```
annotations:`` ``reloader.stakater.com/auto: ``"true"
```

这个向下看，先部署了它。

cat frontend.yaml 



```
kind: Service
apiVersion: v1
metadata:
  name: frontend
spec:
  selector:
    tier: frontend
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  selector:
    matchLabels:
      tier: frontend
  replicas: 1
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: frontend
          image: 172.16.0.12:6166/micro/frontend:latest
          ports:
          - name: http
            containerPort: 80
          volumeMounts:
          - name: frontendconf
            mountPath: /etc/nginx/conf.d/
            readOnly: true
      volumes:
      - name: frontendconf
        configMap:
          name: nginx-frontend
```



这里有个问题就是无法实现热部署，就是更改了配置文件后需要重启Pod,为了处理这个问题github专门有个开源的项目来破解这个问题
下载地址：

```
wget https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
sed -i 's#RELEASE-NAME#config#g' reloader.yaml
kubectl apply -f reloader.yaml
```

当然这个地址必须FQ才能下载，因为这样形式的下载又被墙住了，服！
这里粘贴上github.com 直接搜索到了这个项目的yaml文件

```
https://github.com/stakater/Reloader/blob/master/deployments/kubernetes/reloader.yaml
```



```
---
# Source: reloader/templates/clusterrole.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  labels:
    app: reloader-reloader
    chart: "reloader-v0.0.58"
    release: "reloader"
    heritage: "Tiller"
  name: reloader-reloader-role
  namespace: default
rules:
  - apiGroups:
      - ""
    resources:
      - secrets
      - configmaps
    verbs:
      - list
      - get
      - watch
  - apiGroups:
      - "apps"
    resources:
      - deployments
      - daemonsets
      - statefulsets
    verbs:
      - list
      - get
      - update
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - deployments
      - daemonsets
    verbs:
      - list
      - get
      - update
      - patch

---
# Source: reloader/templates/clusterrolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  labels:
    app: reloader-reloader
    chart: "reloader-v0.0.58"
    release: "reloader"
    heritage: "Tiller"
  name: reloader-reloader-role-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: reloader-reloader-role
subjects:
  - kind: ServiceAccount
    name: reloader-reloader
    namespace: default

---
# Source: reloader/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: reloader-reloader
    chart: "reloader-v0.0.58"
    release: "reloader"
    heritage: "Tiller"
    group: com.stakater.platform
    provider: stakater
    version: v0.0.58
    
  name: reloader-reloader
spec:
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: reloader-reloader
      release: "reloader"
  template:
    metadata:
      labels:
        app: reloader-reloader
        chart: "reloader-v0.0.58"
        release: "reloader"
        heritage: "Tiller"
        group: com.stakater.platform
        provider: stakater
        version: v0.0.58
        
    spec:
      containers:
      - env:
        image: "stakater/reloader:v0.0.58"
        imagePullPolicy: IfNotPresent
        name: reloader-reloader
        args:
      serviceAccountName: reloader-reloader

---
# Source: reloader/templates/role.yaml


---
# Source: reloader/templates/rolebinding.yaml


---
# Source: reloader/templates/service.yaml

---
# Source: reloader/templates/serviceaccount.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: reloader-reloader
    chart: "reloader-v0.0.58"
    release: "reloader"
    heritage: "Tiller"
  name: reloader-reloader
```



 使用方法

如果某deployment需要随着configmap的更新而自动重启pods
只需要添加注释reloader.stakater.com/auto: "true"即可：



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {APP_NAME}-deployment
  annotations:
    reloader.stakater.com/auto: "true"
```



安装ingress-nginx-controller，直接粘贴配置文件
这里先要手动用docker去拉取镜像并且更改名字,还有最新的版本当前是0.30.0，配置文件最新的也有很多的改动，我这里用的旧的版本旧的配置文件，新版本请自行研究
拉取镜像：

```
docker pull registry.aliyuncs.com/google_containers/nginx-ingress-controller:0.26.1
```

更改镜像名字

```
docker tag registry.aliyuncs.com/google_containers/nginx-ingress-controller:0.26.1 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1
```

 

操作完之前的命令后直接创建下面的yaml，创建之前说明下增加了2段

```
# 选择对应标签的node
nodeSelector:
isIngress: "true"
# 使用hostNetwork暴露服务
hostNetwork: true
```

在要创建前端项目的机器上打上标签

```
kubectl label node node-1 isIngress="true"
```

mandatory.yaml



```
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
 
---
 
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
 
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
 
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update
 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get
 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx
 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx
 
---
 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      # wait up to five minutes for the drain of connections
      terminationGracePeriodSeconds: 300
      serviceAccountName: nginx-ingress-serviceaccount
      nodeSelector:
        isIngress: "true"
      hostNetwork: true
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown
 
---
```



查看状态

```
kubectl get pod -n ingress-nginx
NAME                                      READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-bc494d9b-tw8xj   1/1     Running   0          149m
```

创建 ingress 资源

frontend-ingress.yaml



```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-frontend
  annotations:
    kubernets.io/ingress.class: "nginx"
spec:
  rules:
  - host: microtest.XXX.com    //备案过的域名
    http:
      paths:
      - path:
        backend:
          serviceName: frontend   //前端的资源的名字
          servicePort: 80
```



查看目录里面的内容

```
cd /data/frontend/
ls
dist  Dockerfile  frontend.conf  frontend-ingress.yaml  frontend.yaml  mandatory.yaml  reloader.yaml
```

验证
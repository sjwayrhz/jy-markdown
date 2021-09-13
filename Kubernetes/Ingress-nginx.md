# Ingress-nginx 的部署



这个项目在github的地址为

```
https://github.com/kubernetes/ingress-nginx
```

今天是2021年9月10日教师节，翻开一直没有研究的ingress-nginx一看，刚好它发布了v1.0.0版本，很有幸，赶上了第一个正式的大版本。

### 步骤

克隆项目到部署机

```bash
~]# git clone git@github.com:kubernetes/ingress-nginx.git
```

裸金属安装方法

```bash
~]# kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
```

进入裸金属部署

```bash
~]# cd ingress-nginx-controller-v1.0.0/deploy/static/provider/baremetal
```

打开deploy.yaml，搜索image ,替换镜像id

```bash
baremetal]# cat deploy.yaml | grep image
          image: willdockerhub/ingress-nginx-controller:v1.0.0
          imagePullPolicy: IfNotPresent
          image: liangjw/kube-webhook-certgen:v1.0
          imagePullPolicy: IfNotPresent
          image: liangjw/kube-webhook-certgen:v1.0
          imagePullPolicy: IfNotPresent
```

如果需要每个节点都暴露端口，可以修改containerPort.hostPort (可选)

```
ports:
            - name: http
              containerPort: 80
              # hostPort: 80	添加node节点暴露80端口
              protocol: TCP
            - name: https
              containerPort: 443
              # hostPort: 443	添加node节点暴露443端口
              protocol: TCP
            - name: webhook
              containerPort: 8443
              # hostPort: 8443	添加node节点暴露8443端口
              protocol: TCP  
```


# helm部署traefik



### Use the Helm Chart

Add Traefik's chart repository to Helm:

```bash
$ helm repo add traefik https://helm.traefik.io/traefik
```

Pull the helm chart to local disk and untar the file, the lastest chart version is 10.9.1.

```bash
$ helm pull traefik/traefik --untar
```

Remove comments for values.yaml

```bash
$ sed -i 's/# nodePort: 32080/nodePort: 32080/g' traefik/values.yaml
$ sed -i 's/# nodePort: 32443/nodePort: 32443/g' traefik/values.yaml
```

Install traefik from local disk

```bash
$ kubectl create ns traefik-v2
# Install in the namespace "traefik-v2"
$ cd traefik
$ helm install --namespace=traefik-v2 traefik .
```

### Exposing the Traefik dashboard

For instance, the dashboard access could be achieved through a port-forward:

```bash
$ kubectl -n traefik-v2 port-forward $(kubectl -n traefik-v2 get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000 --address=$YOUR_IP
```

It can then be reached at: `http://$YOUR_IP/dashboard/`

其中， $YOUR_IP 是变量，本次执行的机器是 `10.225.63.50`，于是访问地址是`http://10.225.63.50:9000/dashboard/`

Another way would be to apply your own configuration, for instance, by defining and applying an IngressRoute CRD :

```yaml
# dashboard.yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: dashboard
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`traefik.sjhz.tk`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
```

安装方法

```bash
$ kubectl -n traefik-v2 apply -f dashboard.yaml
```

域名是可以自定义的，本次的域名 `traefik.sjhz.tk`解析到的IP地址就是`10.225.63.50`,于是访问的地址是

```
http://traefik.sjhz.tk:32080/dashboard/
```




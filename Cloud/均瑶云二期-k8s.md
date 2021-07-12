### 搭建笔记

K8s-master-01 = 192.168.177.61

K8s-master-02 = 192.168.177.62

K8s-master-03 = 192.168.177.63

```bash
kubeadm init \
  --apiserver-advertise-address=192.168.177.60 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.2 \
  --service-cidr=10.7.0.0/16 \
  --pod-network-cidr=10.3.0.0/16
```

创建记录

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.177.60:6443 --token 83k26v.m7xcknmqfpagx05q \
	--discovery-token-ca-cert-hash sha256:356a418c4258574c5297ef76e460aefaf71afba89be5f2c71be4cf8002da42bc
```

挂载目录

```
cat /etc/fstab

192.168.177.69:/data        /data                    nfs     defaults              0 0
```

生成证书

```
~]# kubeadm token create --print-join-command

~]# kubeadm init phase upload-certs --upload-certs
```

九州云访问科创公钥 位于 192.168.177.4/tmp/id_rsa.pub

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuYCtvG8rZH9w4upBF/ZKF6grzgH8sgBfyYGcZdsg4JMNnQT2UTDLq+JAm00Hm53nQYsHn7Kbezu7NKPCfmHY1dgW1Jbnk8UVOsJEfQzpDu/5GtWj+Lz/UuKb7lu8nQ9V7oC9dJUFtc8hl7EU2DwBlpeSx8xPo+8JOVmXIVt6qqfP1r1F0aRDKQwBU/mEjLRtdnC/qS/ocZO6iIVpuHiQtKD5y9OwKNKbTPL9cZEHVXULdm3E/NwKCFqOTRXeUvOacm2MlA3FC6TBEL6/HqN0eReWgjPoZoBqdJcm0CFfR6WJ4fxb75u3keBvQg5pGTLh4KVa3zHrPDaWgDECgpt2Z root@jh-ping-test
```


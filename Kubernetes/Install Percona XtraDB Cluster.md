# Install Percona XtraDB Cluster on Kubernetes

From : `https://www.percona.com/doc/kubernetes-operator-for-pxc/kubernetes.html`

First of all, clone the percona-xtradb-cluster-operator repository:
```bash
$ git clone -b v1.10.0 https://hub.fastgit.org/percona/percona-xtradb-cluster-operator
$ cd percona-xtradb-cluster-operator/deploy
```
Note

It is crucial to specify the right branch with -b option while cloning the code on this step. Please be careful.

Now Custom Resource Definition for Percona XtraDB Cluster should be created from the deploy/crd.yaml file. Custom Resource Definition extends thstandard set of resources which Kubernetes “knows” about with the new items (in our case ones which are the core of the operator).

This step should be done only once; it does not need to be repeated with the next Operator deployments, etc.
```bash
$ kubectl apply -f crd.yaml
```
The next thing to do is to add the pxc namespace to Kubernetes, not forgetting to set the correspondent context for further steps:
```bash
$ kubectl create namespace pxc
```
In the office document, the default namespaces will be set for pvc.
`kubectl config set-context $(kubectl config current-context) --namespace=pxc`
But I don't want to change defult namespace ,use `-n pxc` instead.
Now RBAC (role-based access control) for Percona XtraDB Cluster should be set up from the deploy/rbac.yaml file. Briefly speaking, role-based access ibased on specifically defined roles and actions corresponding to them, allowed to be done on specific Kubernetes resources (details about users and rolecan be found in Kubernetes documentation).

```bash
$ kubectl -n pxc apply -f rbac.yaml 
```
Note

Setting RBAC requires your user to have cluster-admin role privileges. For example, those using Google Kubernetes Engine can grant user needed privilegewith the following command: $ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value coraccount)

Finally it’s time to start the operator within Kubernetes:
```bash
$ kubectl -n pxc apply -f operator.yaml
```
Now that’s time to add the Percona XtraDB Cluster Users secrets to Kubernetes. They should be placed in the data section of the deploy/secrets.yaml filas logins and plaintext passwords for the user accounts (see Kubernetes documentation for details).

edit secrets password if you want.

```bash
$ cat secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-cluster-secrets
type: Opaque
stringData:
  root: 2wsx#EDC
  xtrabackup: backup_password
  monitor: monitory
  clustercheck: clustercheckpassword
  proxyadmin: admin_password
  pmmserver: admin
  operator: operatoradmin
  replication: repl_password
```

After editing is finished, users secrets should be created using the following command:
```bash
$ kubectl -n pxc create -f secrets.yaml
```
More details about secrets can be found in Users.

Now certificates should be generated. By default, the Operator generates certificates automatically, and no actions are required at this step. Still, yocan generate and apply your own certificates as secrets according to the TLS instructions.

After the operator is started and user secrets are added, change storageClassName for `rook-cephfs`

```shell
#Before changed#

$ cat cr.yaml | grep storageClassName
#        storageClassName: standard
#        storageClassName: standard
#            storageClassName: standard

#After changed#

$ cat cr.yaml | grep storageClassName
        storageClassName: rook-cephfs
        storageClassName: rook-cephfs
            storageClassName: rook-cephfs
```

 Percona XtraDB Cluster can be created at any time with the following command:

```bash
$ kubectl -n pxc apply -f cr.yaml
```
Creation process will take some time. The process is over when both operator and replica set pod have reached their Running status:
```bash
NAME                                               READY   STATUS    RESTARTS   AGE
cluster1-haproxy-0                                 2/2     Running   0          6m17s
cluster1-haproxy-1                                 2/2     Running   0          4m59s
cluster1-haproxy-2                                 2/2     Running   0          4m36s
cluster1-pxc-0                                     3/3     Running   0          6m17s
cluster1-pxc-1                                     3/3     Running   0          5m3s
cluster1-pxc-2                                     3/3     Running   0          3m56s
percona-xtradb-cluster-operator-79966668bd-rswbk   1/1     Running   0          9m54s
```
Check connectivity to newly created cluster
```bash
$ kubectl -n pxc run -i --rm --tty percona-client --image=percona:8.0 --restart=Never -- bash -il
percona-client:/$ mysql -h cluster1-haproxy -uroot -proot_password
```
This command will connect you to the MySQL monitor.
```
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1976
Server version: 8.0.19-10 Percona XtraDB Cluster (GPL), Release rel10, Revision 727f180, WSREP version 26.4.3

Copyright (c) 2009-2020 Percona LLC and/or its affiliates
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

Create a node-port for cluster1

```bash
$ kubectl apply -f pxc-nodeport.yaml

$ kubectl -n pxc get svc
```


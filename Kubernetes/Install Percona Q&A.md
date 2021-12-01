FROM: `https://blog.csdn.net/sinat_32582203/article/details/119827093`
(1) 如何在其他namespace中访问pxc？

可以通过FQDN访问pxc，例如，本示例中的FQDN为cluster1-haproxy.pxc.svc.cluster.local 或者 cluster1-haproxy-replicas.pxc.svc.cluster.local

(2) 两个服务cluster1-haproxy和cluster1-haproxy-replicas有什么区别？

cluster1-haproxy服务默认连接cluster1-pxc-0，如果cluster1-pxc-0不可用，则按降序依次选择其余节点，例如cluster1-pxc-2，cluster1-pxc-1。

cluster1-haproxy-replicas则采用轮询方法，依次请求各个节点。

(3) 如何进入pxc容器内部？

每个cluster1-pxc pod包含一个pxc容器，为数据库所在容器，可直接进入操作数据库（不建议）：

    $ kubectl exec -it cluster1-pxc-0 -n pxc --container pxc -- /bin/bash
    bash-4.4$ mysql -uroot -proot_password
    mysql: [Warning] Using a password on the command line interface can be insecure.
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 10728
    Server version: 5.7.33-36-57 Percona XtraDB Cluster (GPL), Release rel36, Revision a1ed9c3, WSREP version 31.49, wsrep_31.49
     
    Copyright (c) 2009-2021 Percona LLC and/or its affiliates
    Copyright (c) 2000, 2021, Oracle and/or its affiliates.
     
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
     
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
     
    mysql> 

(4) 如何查看数据库root密码？

数据库密码以base64编码保存在secret中，可通入如下方法查看root密码：

    $ kubectl get secret -n pxc
    NAME                                          TYPE                                  DATA   AGE
    default-token-fch4c                           kubernetes.io/service-account-token   3      16h
    internal-cluster1                             Opaque                                8      96m
    my-cluster-secrets                            Opaque                                8      16h
    my-cluster-ssl                                kubernetes.io/tls                     3      96m
    my-cluster-ssl-internal                       kubernetes.io/tls                     3      96m
    percona-xtradb-cluster-operator-token-cktgk   kubernetes.io/service-account-token   3      16h
     
    $ kubectl get secret my-cluster-secrets -n pxc -o yaml 
    apiVersion: v1
    data:
      clustercheck: Y2x1c3RlcmNoZWNrcGFzc3dvcmQ=
      monitor: bW9uaXRvcnk=
      operator: b3BlcmF0b3JhZG1pbg==
      pmmserver: YWRtaW4=
      proxyadmin: YWRtaW5fcGFzc3dvcmQ=
      replication: cmVwbF9wYXNzd29yZA==
      root: cm9vdF9wYXNzd29yZA==
      xtrabackup: YmFja3VwX3Bhc3N3b3Jk
    kind: Secret
    metadata:
      creationTimestamp: "2021-08-19T09:58:34Z"
      name: my-cluster-secrets
      namespace: pxc
      resourceVersion: "2432784"
      uid: bef2dec9-4f11-494a-ad06-925b579fa418
    type: Opaque

得到数据库root密码的base64编码为cm9vdF9wYXNzd29yZA==，解码即可：

    $ echo 'cm9vdF9wYXNzd29yZA==' | base64 -d
    root_password

(5) 如何导入已有数据库到pxc中？

暂时只想到了一个方法。在上述安装过程中，可以发现，hostpath卷挂载在容器的/var/lib/mysql。所以可以将从别的数据库导出的.sql文件放在节点的/data/pxc/目录下，然后按第三步进入到容器内部，执行导入命令。

(6) 使用过程中发现percona的一些自定义资源无法正确删除？

可以通过编辑资源的yaml文件，将metadata的finalizers字段置空来强制删除，例如：

kubectl patch crd/perconaxtradbclusters.pxc.percona.com -p '{"metadata":{"finalizers":[]}}' --type=merge

(7) 如何查看备份文件？

    $ kubectl get pxc-backup -n pxc
    NAME                                           CLUSTER    STORAGE   DESTINATION                                     STATUS      COMPLETED   AGE
    cron-cluster1-fs-pvc-20218207400-1nhng         cluster1   fs-pvc    pvc/xb-cron-cluster1-fs-pvc-20218207400-1nhng   Succeeded   39m         40m

备份文件保存在DESTINATION列的pvc中
————————————————
版权声明：本文为CSDN博主「洒满阳光的午后」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/sinat_32582203/article/details/119827093
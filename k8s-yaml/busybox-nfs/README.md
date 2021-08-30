这是一个关于busybox创建pv,pvc的案例，需要满足以下条件
- 完成了nfs服务器的搭建
- nfs服务器上需要搭建nfs-client-provisioner
- 创建的storageclass的名字是nfs-client
满足以上条件可以使用该yaml，当然也可以自行修改sc的名字。
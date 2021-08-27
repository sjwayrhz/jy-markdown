### 发布脚本

```
case ${OPERATION} in 
    deploy)
        /docker/kubernets/dockerlogin/login.sh &&  source /etc/profile   && cd  $WORKSPACE/Sole_Spider && pwd && ls && ./build_k8s.sh && /var/lib/jenkins/Parameter/write_verison.sh ${JOBNAME}
        ;;
    rollback)
 cd  /docker && pwd &&  source /etc/profile  && ./Last_edition.sh ${JOBNAME} ${VERSION}
        ;;
```

### login.sh

```
~]# cat /docker/kubernets/dockerlogin/login.sh
docker login -u cn-east-2@XLBUARYJX0ZWG7U8CCON -p dfa7112fcf2254c5a0d0c40b932f82ce426a21e018202738f4b1f18c19001f11 swr.cn-east-2.myhuaweicloud.com
```

### /etc/profile中末尾添加的内容

```
export JENKINS_USER=root
export JENKINS_GROUP=root
export JENKINS_HOME=/root/.jenkins
export KTHWTEST=/etc/k8s/owl_test.json


export XKDPC=/etc/k8s/xingkuang_k8s_prod.json


alias XKDPC='kubectl --kubeconfig /etc/k8s/xingkuang_k8s_prod.json'
```
### build.k8s.sh

```
#!/usr/bin/env bash
re="100.125.17.64:20202/lanjing/([^:]+):([^ ]+)"
imageStr=$(kubectl --kubeconfig $KTHWTEST get deploy paw-sole-spider -o jsonpath='{..image}')
echo "current version"

if [[ $imageStr =~ $re ]]; then echo ${BASH_REMATCH[2]}; fi

if [ $# -eq 0 ];
then nextVersion=$(./increment_version.sh -p ${BASH_REMATCH[2]});
else nextVersion=$(./increment_version.sh $1 ${BASH_REMATCH[2]});
fi

echo "next version"
echo $nextVersion;


docker build -t swr.cn-east-2.myhuaweicloud.com/lanjing/paw-sole-spider:$nextVersion .
docker push swr.cn-east-2.myhuaweicloud.com/lanjing/paw-sole-spider:$nextVersion

kubectl --kubeconfig $KTHWTEST set image deployment/paw-sole-spider paw-sole-spider=100.125.17.64:20202/lanjing/paw-sole-spider:$nextVersion
kubectl --kubeconfig $KTHWTEST get pod -l app=paw-sole-spider
```

### Last_edition.sh内容

```
~]# cat /docker/Last_edition.sh
#!/usr/bin/env bash
deployment=$1
nowversion=$2
echo "canshu ${deployment} - ${nowversion}"

re="100.125.17.64:20202/lanjing/([^:]+):([^ ]+)"
imageStr=$(kubectl --kubeconfig $KTHWTEST get deploy $deployment -o jsonpath='{..image}')
echo "current version"
if [[ $imageStr =~ $re ]]; then echo ${BASH_REMATCH[2]}; fi


if [[ ${nowversion} == "0.0.0" ]]; then
echo "=="$nowversion
 nextVersion=$(./increment_version.sh -p ${BASH_REMATCH[2]});
else
echo "<>"$nowversion
 nextVersion=$nowversion;
fi

echo "next version"
echo $nextVersion;

# git add .
# git commit -m $nextVersion --no-verify
# git push

#docker build -t swr.cn-east-2.myhuaweicloud.com/lanjing/paw-cron:$nextVersion .
#docker push swr.cn-east-2.myhuaweicloud.com/lanjing/paw-cron:$nextVersion

echo --kubeconfig $KTHWTEST set image deployment/$deployment $deployment=100.125.17.64:20202/lanjing/$deployment:$nextVersion

kubectl --kubeconfig $KTHWTEST set image deployment/$deployment $deployment=100.125.17.64:20202/lanjing/$deployment:$nextVersion
kubectl --kubeconfig $KTHWTEST get pod -l app=$deployment
```


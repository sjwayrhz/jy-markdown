bbb语音通话不可以用

```
从云空间（http://lms-shs.wfl-ischool.cn）里面的直播，会跳转到（https://bbb-shs.wfl-ischool.cn），然后点击麦克风，会出现呼叫失败的问题
```

登录bbb所在的均瑶云服务器10.220.80.6
```
ssh -i K8s-tools/id_rsa.liukun root@10.220.80.6
```

添加公网ip
```
ip addr add 103.192.254.13/32 dev ens3
```

如果需要重启docker-compose
```
dc stop && dc up -d
```

参考的github项目
```
https://github.com/bigbluebutton/docker/blob/develop/docs/behind-nat.md
```
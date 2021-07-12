# EUserv服务器登记

服务器开启访问ipv4
```

echo -e "nameserver 2001:67c:2b0::4\nnameserver 2001:67c:2b0::6" > /etc/resolv.conf

~]# cat /etc/resolv.conf nameserver 2001:67c:2b0::4 nameserver 2001:67c:2b0::6
```

还有其它的dns
```
++

totd.aa.net.uk (2001:8b0:6464::1)

++

nameserver 2001:67c:27e4:15::6411 nameserver 2001:67c:27e4::64
```

e@sjhz.cf
```
k8s-master

e.sjhz.ga

vServer VS2-free v2.1 -- Centos 8.2

Servername: srv17050.blue.kundencontroller.de

IPv6: 2a02:0180:0006:0001:0000:0000:0000:2f3a

root / YFNDGjL7
```
f@sjhz.cf 
```
k8s-node1

f.sjhz.ga

vServer VS2-free v2.1 -- Centos 8.2

Servername: srv17091.blue.kundencontroller.de

IPv6: 2a02:0180:0006:0001:0000:0000:0000:2f5d

root / 4yO80Yg1
```
g@sjhz.cf
```
k8s-node2

g.sjhz.ga

vServer VS2-free v2.1 -- Centos 8.2

Servername: srv17064.blue.kundencontroller.de

IPv6: 2a02:0180:0006:0001:0000:0000:0000:2f44

root / 94BsIg39

===============================================================================
```
公钥

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxHNFADkxs2KZJGdR6J4DZM3mlKMwi75j4sHDu3SE2mNc1MZJ/B/vkxdYeLfChFxNPTW628zbNiCkFySDKb6kGmlzBam9OKNWc5zNnRlp61Jt3PSoBUvZWHHXXljptr273mkQ4LRnk9tyJZlItagvE9mS36uPuDB11tP8S6VkabvFGj8qIPrj9Uqv0brQlOApgwnzJ44oSb7CNECdiTnT8jFfU++UhT6MqGDgsOJL9AkIiC7IizmTlLVilDWIDKb/ndRYng3KaNEh+0an0i/feBYh3uYULxGWmFQQnqwqfx+zZR0uvvZ/KvW0bt7CbBEPIFQFv1rgcjPP9hVWWGPzHOygooWtwyLiIVIO38HOZwonX/nAOCOmYW6f1rczz8PzlHm0nTLyQA5kmJ0biv8rWC8oA8uXNw3jVcQDlE3F3XQ3uEUjXQTfRROxqNV3P4JklVCfbRcf1ASMwep7Cj2EQHgs7umXAWIq4ToZB7tf4qK8I42xBTxo+gYK0lNi0bRc= root@srv17050
```



私钥

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```


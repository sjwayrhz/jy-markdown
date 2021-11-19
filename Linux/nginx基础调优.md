### nginx 报错 upstream timed out (110: Connection timed out)解决方案

nginx作为反向代理服务器时，报错： upstream timed out (110: Connection timed out)……

经过百度，google看到都是修改nginx配置，解决超时问题，比如：
``` 
large_client_header_buffers 4 16k;
client_max_body_size 300m;
client_body_buffer_size 128k;
proxy_connect_timeout 600;
proxy_read_timeout 600;
proxy_send_timeout 600;
proxy_buffer_size 64k;
proxy_buffers   4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```

但是这些都是设置缓存或者超时长度的，根本不能解决上游服务器upstream，响应慢的问题，最后通过google看到如此说的：

最终通过这两个设置，设置http版本和header的Connection来解决该问题：
```
proxy_http_version 1.1;
proxy_set_header Connection "";
```

设置这两项就能解决，但是不知道为啥，后续再探究，有懂的欢迎赐教！

## nginx 出错:socket() failed (24: Too many open files) while connecting to upstream

1. 错误描述

通过nginx负载两个节点的rabbitmq
当用java代码创建超过500个连接时(我的机器默认只能创建这么多)，出现错误：
```
com.rabbitmq.client.ShutdownSignalException: connection error
java.net.SocketException: Software caused connection abort: recv failed
```

查看nginx日志/var/log/nginx/error.log，发现错误
socket() failed (24: Too many open files) while connecting to upstream
解决

    修改linux打开文件句柄数，编辑vi /etc/security/limits.conf，添加
```
<domain>      <type>  <item>         <value>
*             soft   nofile          204800
*             hard   nofile          204800
```


    修改nginx打开文件数, 编辑nginx.conf，添加worker_rlimit_nofile值

```
worker_processes  1;
worker_rlimit_nofile 20480;
```

重启nginx后问题解决

网络流量的抓包可以使用tcpdump命令

例如使用如下命令，可以抓取jenkins的网络流量

```
tcpdump -i any port 8080 -nne -vv -s0  -w /tmp/jenkins.pcap
```

然后可以通过wireshark分析


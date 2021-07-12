Mac使用Cisco anyconnect报错

抱错内容：AnyConnect cannot confirm it is connected to your secure gateway. The local network may not be trustworthy. Please try another network.

解决办法

```
cd /opt/cisco/AnyConnect
sudo nano AnyConnectLocalPolicy.xml
然后编辑 ExcludeMacNativeCertStore 选项把false 改成 "true"<ExcludeMacNativeCertStore>true</ExcludeMacNativeCertStore>
^X (control X 保存) 按 Y 退出
```

然后重启cisco vpn即可

其中,均瑶云cisco账号

```
vpn地址
116.228.40.4
用户名：jykcgy01
密码 ：20210208#Kc
组名称：ssl-juneyao-75

用户名称：	jykc02
用户密码：	K20210322#c		
组名称：	ssl-juneyao-75
```



登陆均瑶云 `https://10.220.30.100/auth/login/?next=/`的时候发现https安全性问题

原因：因为Chrome不信任这些自签名ssl证书，为了安全起见，直接禁止访问了，thisisunsafe 这个命令，说明你已经了解并确认这是个不安全的网站，你仍要访问就给你访问了。
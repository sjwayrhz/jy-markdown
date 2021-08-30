## Mac下访问github网址速度慢或者无法访问终极解决方案

### 第一步，进入host文件

```text
sudo vim /etc/hosts
```

### 第二步，进入编辑模式

按E键，然后回车。

### 第三步，启动编辑模式

按I键，启动编辑模式。

### 第四步，把以下命令添加到host文件内

使用光标移动到空白处，复制以下命令，粘贴到空白处。

```text
# Github
151.101.185.194 github.global.ssl.fastly.net
140.82.114.4 github.com 
151.101.112.133 assets-cdn.github.com 
151.101.184.133 assets-cdn.github.com 
185.199.108.153 documentcloud.github.com 
192.30.253.118 gist.github.com
185.199.108.153 help.github.com 
192.30.253.120 nodeload.github.com 
151.101.112.133 raw.github.com 
23.21.63.56 status.github.com 
192.30.253.1668 training.github.com 
192.30.253.112 www.github.com 
151.101.13.194 github.global.ssl.fastly.net 
151.101.12.133 avatars0.githubusercontent.com 
151.101.112.133 avatars1.githubusercontent.com
```

**注意：** 目前都是本地hosts配置了[http://github.com](https://link.zhihu.com/?target=http%3A//github.com)的ip地址，如果访问github失败，或者访问网速慢，可能就是github的ip地址换了或者ip地址丢包严重。可以通过`ping github.com`查看时长以及丢包率。

如果需要修改github ip地址，可以通过[https://github.com.ipaddress.com](https://link.zhihu.com/?target=https%3A//github.com.ipaddress.com/) ，了解当前github.com的ip地址。

### 第五步，退出编辑模式

按esc键。

### 第六步，保存并退出host文件

键入以下命令。

```text
:wq
```

### 第七步，刷新DNS

键入以下命令。

```text
dscacheutil -flushcache
```

### 第八步，测试github网址

[https://github.com/](https://link.zhihu.com/?target=https%3A//github.com/)
# vmware esxi 导出ova

[TOC]

### 资源准备

#### 下载

官方下载链接

```
https://developer.vmware.com/web/tool/12.5.0/vmware-powercli
```

下载文件“VMware-PowerCLI-12.5.0-19195797.zip”

蓝奏云下载路径

```
https://wwi.lanzoup.com/iX2lf02hglbi
```

#### 解压

win10解压到以下路径

```
C:\Program Files\WindowsPowerShell\Modules
```

### 配置PowerShell

远程执行策略为允许

打开powershell执行以下命令

```powershell
Set-ExecutionPolicy RemoteSigned
```

忽略证书验证，需要等待大约15s

```powershell
Set-PowerCLIConfiguration -Scope AllUsers -ParticipateInCeip $false -InvalidCertificateAction Ignore
```

至此powershell环境配置完毕

### 连接ESXI

使用Connect-VIServer命令连接ESXI

Connect-VIServer 虚拟机IP或主机名

例如 本次连接 10.220.62.247，需要等待大约30s，然后输入esxi的账号密码

```powershell
Connect-VIServer 10.220.62.247
```

会有如下输出

```
Name                           Port  User
----                           ----  ----
10.220.62.247                  443   root
```

看到这个则表示连接成功

查看这个esxi中包含的虚拟机

```powershell
get-vm
```

### 导出OVA

命令模板

```powershell
Get-VM -Name $esxi_hostname | Export-VApp -Destination '$windows_dir' -Format OVA
```

举例

```powershell
Get-VM -Name wflps-gitlab-58 | Export-VApp -Destination 'D:\ova_back' -Format OVA
```

如果关闭了powershell, 则需要重新连接 esxi server
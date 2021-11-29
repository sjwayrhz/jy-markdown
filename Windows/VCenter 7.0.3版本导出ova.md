## 先安装vmware powerCLI

下载vmware-powercli 工具地址如下
```
https://code.vmware.com/web/tool/11.5.0/vmware-powercli
```
下载后这个是离线包

### win10
解压到以下路径 `C:\Program Files\WindowsPowerShell\Modules`

### win7
解压到 `C:\Windows\System32\WindowsPowerShell\v1.0\Modules`

但是win7的版本太低了所以要升级
```
下载安装dotNetFx40_Full_x86_x64.exe

http://go.microsoft.com/fwlink/?linkid=247962

下载安装更新powershell包

http://download.microsoft.com/download/E/7/6/E76850B8-DA6E-4FF5-8CCE-A24FC513FD16/Windows6.1-KB2506143-x64.msu

然后重启win7主机查看是否更新到3的版本

Get-Host | Select-Object Version
```
## 配置powershell

### 配置远程执行策略为允许

打开powershell执行以下命令（WIN10的话输入A）
```
Set-ExecutionPolicy RemoteSigned
```
### 配置忽略证书验证
命令如下这里无论是win10还是win7都输入A
```
Set-PowerCLIConfiguration -Scope AllUsers -ParticipateInCeip $false -InvalidCertificateAction Ignore
```
至此powershell环境配置完毕

使用Connect-VIServer命令连接VC
```
Connect-VIServer 虚拟机IP或主机名
```
然后输入用户名密码

可以get-vm查看虚拟机

### 连接完成导出OVA格式
(需要注意的是导出之前要删除所有快照以及挂载的IOS镜像（如有）)

命令如下
```
Get-VM -Name 主机名 | Export-VApp -Destination 'D:\' -Format OVA
```
命令详解释  Get-VM -Name 主机名 | Export-VApp -Destination '存放路径' -Format OVA


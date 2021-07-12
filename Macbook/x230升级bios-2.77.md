**X230系列白名单（免编程器）**



详细操作


1、首先按照坛友ag1332所述先对BIOS进行设置：Security->UEFI BIOS Update Option-> Flash BIOS Updating by End-User <Enable>; Secure RollBack Prevention <Disabled>。 SecureBoot选Disable

2、将下载的1vyrain.iso，用Rufus写入U盘，用RUFUS时选择1vyrain ISO ,勾选 GPT分区模式，开始制作启动盘；提示中选择DD模式，RUFUS刷入完成后，WIN系统会提示U盘需格式化，直接点取消。

3、关机（注意不是重新启动，一定是先关机，然后再启动，否则电脑不会进入U启动，具体原因不详，有待探讨），稍后按电源键开机，按F12选择U盘启动。
4、 进入1vyrain画面，画面提示ENTER确认操作（见下图所示），ENTER后黑屏，电源灯在间断的闪烁，再按电源键，屏幕开启（见下图所示），在屏幕的最下端提示有4个条目0、1、2、3共4条可供选择，选择条目1 （FLASH MODIFIED LENOVO BIOS）【Flash 改进的联想 bios】（输入数字1，按Enter键确认后，正式开始刷BIOS）。刷机中途有几次提示，直接选择默认项按Enter键确认即可，刷机结束后自动重启开机。

为了手上的T430和X230换网卡，黑苹果，入手了编程器，刷了T430和X230。手头还有一台T430S，由于BIOS芯片的封装问题一直没动手。这两天琢磨了一下，居然老外不需要编程器就能刷白名单，我今天试了一下，居然顺利把T430S刷成了白名单的2.76。给大家介绍如下，E文好的可以直接访问下面的链接。

1、首先把BIOS降级到规定的版本（记得清除BIOS密码），记得改一下BIOS的安全选项：Security->UEFI BIOS Update Option-> Flash BIOS Updating by End-User <Enable>; Secure RollBack Prevention <Disabled>。降级方法就是下载IVprep-master.zip，解压到电脑。直接运行根目录下的downgrade.bat （不需要用管理员模式运行）机器会重启，自动降级到对应的低版本。我的好像降到2.59了。（请接好电池和电源！！！）
2、下载1vyrain.iso，用Rufus写入U盘（请使用D模式，写入的时候有提示，选DD模式即可）
3、开机进BIOS把SecureBoot选Disable，并选UEFI启动模式，用刚才写好的U盘启动
4、按照提示信息操作就好，都是英文提示，不复杂。（参考30楼热心坛友的中文解释）
备注：你的笔记本会重启几次并报CRC 安全提示，千万别害怕，重启几次就没了。
5、重启以后你的BIOS会再次升级到不被锁的最高版本，我的T430S升级到了2.76

完工！！！享受一下功能：

1、CPU超频
2、移除白名单
3、高级菜单支持
4、高级菜单禁止Intel ME功能

需要的工具和软件都在这里：(T430S安装了10.15.4，换了DW1510(bcm94322HM8L，功能正常，能有米，换装三根ipex4天线，上bcm94360HMB网卡)

链接：https://pan.baidu.com/s/1nH9T15mmnlaNBmUhuUHuAg
提取码：bpvj


适用机型列表：



- X230
- X330* （改屏机）
- X230t
- T430
- T430s
- T530
- W530



如果你的机型的BIOS版本低于或等于下表列出的版本，可用忽略降级BIOS的步骤，直接用1vyrain.iso刷白名单：

| 机型  | BIOS版本 |
| ----- | -------- |
| X230  | 2.60     |
| X330  | 2.60     |
| X230T | 2.58     |
| T430  | 2.64     |
| T430S | 2.59     |
| T530  | 2.58     |
|       |          |



温馨提示：刷机有危险，后果自负！

参考网页：

```
https://github.com/n4ru/1vyrain

https://1vyra.in/

https://rufus.ie/
```


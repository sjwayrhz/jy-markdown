使用iTerm2 自带的密码管理器输入密码

1. 在密码管理器里面预设密码
   打开 iTerm2, 菜单：Window -> Password Manager 打开密码管理器;

然后点击左下方的 + 号, 然后 添加一个密码

2. 设置 trigger
   打开 iTerm2, 菜单 iTerm2 -> Preference 然后开始设置

Profiles -> 选择你使用的profile,比如Default -> Advanced -> Trigger -> Edit -> 打开 Triggers
右下角 + 号,
Regular Expression -> ssh. *或者 password.*
Action: Open Password Manager 
Parameter: 选择你上一步输入的密码管理器的设置的;

很多情况你可能在第一次ssh 一个新机器的时候会遇到接受认证的消息, 让你接受一个新的认证,给 yes/no, 其实有办法默认接受所有的认证:
编辑 ~/.ssh/config, 若没有,则新建. 改成如下类似, 我这里是如果机器名是 *.tianxiaohui.com, 则默认接受

```
[xiatian@eric.tianxiaohui.com ~]$ cat ~/.ssh/config
Host *.tianxiaohui.com
    StrictHostKeyChecking no
```

如果修改了 ~/.ssh/config 之后, 报下面的错:
Bad owner or permissions on /export/home/xiatian/.ssh/config
则修改这个文件的权限: 
chmod 600 ~/.ssh/config
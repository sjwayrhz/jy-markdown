## iterm2安装lrzsz

[TOC]

由于mac自带的terminal不带终端，于是可以使用iterm2来做terminal。

### 下载iterm2

官网下载链接如下

```
https://iterm2.com/downloads/stable/iTerm2-3_4_3.zip
```

下载之后并安装

### 安装brew

国外

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
国内：
```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### 安装lrzsz

```
brew install lrzsz
```

如果使用上述命令不生效的话，可以使用

```
arch -x86_64 brew install <package>
```

如果必须使用“arch -x86_64”才可以生效，也可以考虑修改home目录下的 .bash_profile

```
 vi .bash_profile
 
 alias brew='arch -x86_64 brew'
```

### 安装脚本到mac指定目录

保存 iterm2-send-zmodem.sh 和 iterm2-recv-zmodem.sh 到mac的 /usr/local/bin/ 路径下
注意添加脚本可执行权限：

```
chmod +x iterm2-send-zmodem.sh
chmod +x iterm2-recv-zmodem.sh
```

### 设置iterm2

*带双引号的都需要点击*

打开iTerm2后，在软件设置菜单栏-“profiles”->"open profiles"->"edit profiles";

然后在"profiles",在 Default中点击“Advanced”,点击Triggers下的“Edit”。

Triggers设置如下：

```
\*\*B010                         Run Silent Coprocess        /usr/local/bin/iterm2-send-zmodem.sh

\*\*B00000000000000              Run Silent Coprocess        /usr/local/bin/iterm2-recv-zmodem.sh
```

新增后点击"Close"即完成配置。

重启iterms2完成配置。

### 使用方法

通过ssh登陆到目的linux虚拟机之后，安装lrzsz

```
Centos:  yum install lrzsz
Ubuntu:  apt install lrzsz
```

安装之后，输入 rz 导入文件，输入 sz 传出文件。 

### shell脚本

iterm2-recv-zmodem.sh

```
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
	FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
	FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi

if [[ $FILE = "" ]]; then
	echo Cancelled.
	# Send ZModem cancel
	echo -e \\x18\\x18\\x18\\x18\\x18
	sleep 1
	echo
	echo \# Cancelled transfer
else
	cd "$FILE"
	/usr/local/bin/rz -E -e -b
	sleep 1
	echo
	echo
	echo \# Sent \-\> $FILE
fi
```

iterm2-send-zmodem.sh

```
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
	FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
	FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
	echo Cancelled.
	# Send ZModem cancel
	echo -e \\x18\\x18\\x18\\x18\\x18
	sleep 1
	echo
	echo \# Cancelled transfer
else
	/usr/local/bin/sz "$FILE" -e -b
	sleep 1
	echo
	echo \# Received $FILE
fi
```





问题：brew install报错

```
Error: Cannot install in Homebrew on ARM processor in Intel default prefix (/usr/local)
```

```
For what it's worth, before installing Homebrew you will need to install Rosetta2 emulator for the new ARM silicon (M1 chip). I just installed Rosetta2 via terminal using:

/usr/sbin/softwareupdate --install-rosetta --agree-to-license

This will install rosetta2 with no extra button clicks.

After installing Rosetta2 above you can then use the Homebrew cmd and install Homebrew for ARM M1 chip:  arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

Once Homebrew for M1 ARM is installed use this Homebrew command to install packages: arch -x86_64 brew install <package>

```


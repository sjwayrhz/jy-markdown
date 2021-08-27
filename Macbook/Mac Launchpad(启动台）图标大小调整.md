# Mac Launchpad(启动台）图标大小调整

**具体方法如下：**

**在Terminal下执行一下命令：**

```
defaults write com.apple.dock springboard-columns -int 9

defaults write com.apple.dock springboard-rows -int 7

defaults write com.apple.dock ResetLaunchPad -bool TRUE

killall Dock
```



**1、调整每一列显示图标数量，10表示每一列显示10个，比较不错，可根据个人喜好进行设置。**

defaults write com.apple.dock springboard-rows -int 10

**2、调整多少行显示图标数量，这里我用的是8** 

defaults write com.apple.dock springboard-rows -int 8

**3、重置Launchpad**	

defaults write com.apple.dock ResetLaunchPad -bool TRUE

**4、重启Dock**

killall Dock

**再次进入Launchpad，美观大方。**



**恢复默认设置的方法。**

**以下是恢复默认大小的命令（在Terminal执行即可）：**

```
defaults write com.apple.dock springboard-rows Default

defaults write com.apple.dock springboard-columns Default

defaults write com.apple.dock ResetLaunchPad -bool TRUE

killall Dock
```


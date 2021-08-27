https://www.zhihu.com/question/374200060/answer/1450386406

defaults write com.apple.systempreferences AttentionPrefBundleIDs 0





按需执行如下代码操作：

忽略大版本更新提示：

```
sudo softwareupdate --ignore "macOS Catalina"
sudo softwareupdate --ignore "macOS Big Sur"
```

忽略小版本更新的方法：

```
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
```

具体根据版本来定

取消小红点：

```
sudo softwareupdate --ignore "macOS Catalina 10.15.4 Update"

killall Dock
```

恢复更新提示：

```
sudo softwareupdate --reset-ignored
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
```


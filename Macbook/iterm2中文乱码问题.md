# iterm2本地中文乱码问题

经过一番摸索后，发现这种情况一般是终端和服务器的字符集不匹配，MacOSX 下默认的是 utf8 字符集。
 输入 locale 可以查看字符编码设置情况，而我的对应值是空的。
 因为我在本地和服务器都用 zsh 替代了 bash，而且使用了 oh-my-zsh，而默认的.zshrc 没有设置为 utf-8 编码，所以本地和服务器端都要在.zshrc 设置，步骤如下，bash 对应.bash_profile 或.bashrc 文件。

1.在终端下输入`vim ~/.zshrc`

或者使用其他你喜欢的编辑器编辑~/.zshrc 文件

2.在文件内容末端添加：

```
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

接着重启一下终端，或者输入`source ~/.zshrc`使设置生效。
 设置成功的话，在本地和登录到服务器输入 `locale` 回车会显示下面内容。

```
LANG="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_CTYPE="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
```

这时，中文输入和显示都正常了。
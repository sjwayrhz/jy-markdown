# Git更改默认私钥

[TOC]

## Linux

密钥名称为gitkey , 放置位置为：`~/.ectw/gitkey`

```bash
$ git config --global core.sshCommand "ssh -i ~/.ectw/gitkey"
```

## Windows

使用git bash工具，用命令行方式创建.ectw文件夹 `mkdir .ectw`,并隐藏该文件夹

密钥名称为gitkey , 放置位置为：`C:\.ectw\gitkey`

```bash
$ git config --global core.sshCommand "ssh -i C:/.ectw/gitkey"
```

## MACOS

密钥名称为gitkey , 放置位置为：`~/.ectw/gitkey`

一般情况下使用的是普通用户登录，所以需要更改文件权限

```bash
$ sudo chmod 660 ../.ectw/gitkey
```

然后设置git

```bash
$ git config --global core.sshCommand "ssh -i ~/.ectw/gitkey"
```




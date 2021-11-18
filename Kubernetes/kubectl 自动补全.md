安装 bash-completion

```
$ dnf install bash-completion -y
```

添加环境变量

```
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
$ source ~/.bashrc
```


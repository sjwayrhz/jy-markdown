```
[WARNING]: Platform linux on host k8s-node-08 is using the discovered Python interpreter at /usr/libexec/platform-python, but future installation of another Python interpreter could
change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
```



全局配置

在ansible.cfg的[defaults]部分添加配置

interpreter_python = 选项，例如

interpreter_python = auto_legacy_silent
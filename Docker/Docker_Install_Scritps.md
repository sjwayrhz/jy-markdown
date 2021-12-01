可以通过github的链接脚本安装
```bash
~]# wget -O- https://gitee.com/sjwayrhz/one_key_install/raw/master/install_docker.sh | sh
```


一键安装脚本如下

```shell
#!/bin/bash

echo -e "\033[32m  指定安装的docker版本,例如"-20.10.6-3.el7",可在加入yum源后通过yum list docker-ce --showduplicates|sort -r查看 \033[0m"
docker_version=""

echo -e "\033[32m  安装必要的一些系统工具 \033[0m"
sudo yum install yum-utils device-mapper-persistent-data lvm2 -y

echo -e "\033[32m  添加软件源信息 \033[0m"
sudo wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

echo -e "\033[32m  更新yum源并安装Docker-CE \033[0m"
sudo yum install docker-ce docker-ce-cli -y

echo -e "\033[32m  修改daemon.json配置 \033[0m"
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://wlainhm4.mirror.aliyuncs.com"],
  # "graph": "/opt/docker",
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

echo -e "\033[32m  生成阿里云登录密钥 \033[0m"
sudo mkdir ~/.docker
sudo tee ~/.docker/config.json <<-'EOF'
{
	"auths": {
		"registry.cn-shanghai.aliyuncs.com": {
			"auth": "dGFvaXN0bW9ua0AxNjMuY29tOlZ3djU2dHk3"
		}
	}
}
EOF
echo -e "\033[32m  重启docker引擎服务 \033[0m"
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker

echo -e "\033[32m  检查docker是否安装成功 \033[0m"
if [ $? -eq 0 ];then
    echo -e "\033[1;32m Docker start successful. \033[0m"
else
    echo -e "\033[1;31m Docker start failed!please check! \033[0m"
fi
```
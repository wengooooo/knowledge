
## Docker安装

## 系统要求

 - centos 7
 - docker 19.03

## 安装docker 19.03 rpm
如果需要其他版本，去[docker官网](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
下载即刻，但是必须三个rpm是版本一致的
```
# cd /usr/local/src
# wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-cli-19.03.9-3.el7.x86_64.rpm
# wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
# wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-19.03.9-3.el7.x86_64.rpm

# yum install containerd.io-1.2.6-3.3.el7.x86_64.rpm
# yum install docker-ce-cli-19.03.9-3.el7.x86_64.rpm
# yum install docker-ce-19.03.9-3.el7.x86_64.rpm

```

## 启动docker

```
# systemctl enable docker
# systemctl start docker
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4MDk4NDAzNzBdfQ==
-->
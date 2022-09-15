# v2ray 服务器安装

登入服务器

```bash
bash <(curl -s -L https://git.io/v2ray.sh)
```

如果无法下载脚本，使用文件夹的脚本

然后一路next下去即可



客户端

windows下载v2rayNG



ios 下载 shadowrocket





注意

连不上v2ray就关闭服务器的防火墙

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

netstat: command not found

```
yum install net-tools
```



查看是否启动

```
v2ray.service - V2Ray Service
```

查看监听情况

```
netstat -apn | grep v2ray
```

如果没有看见绑定公网，手动绑定一下

https://www.linodovultr.com/post/resolve-v2ray-after-install-can-not-connect.html?replytocom=59
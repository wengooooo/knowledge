



# Postfix Dovecot Sieve
工作流程
 - Postfix 收到邮件传递给dovecot 
 - dovecot启动sieve进行过滤
 - sieve启用sieve extprograms传递给pshell
 - shell调用Python
 - python做其他事情

# 系统配置

 - 腾讯云
 - centos 7
 - postfix 2.10.1 
 - dovecot 2.3
 ```
 #查看版本
 # postconf -d | grep mail_version
 # rpm -qa | grep postfix
 # dovecot --version
 ```
 

## 安装postfix
```
# yum install postfix
```

## 配置postfix
```
# vi /etc/postfix/main.cf
# :set nu
# 第76行邮局主机名修改
#myhostname = virtual.domain.tld => myhostname = ns
# 第83行邮件域名修改
#mydomain = domain.tld =>  mydomain = yixie4.org
# 第99行的发送接收邮件域名（已定义把#去掉就好了）
#myorigin = $mydomain => myorigin = $mydomain
# 第116行的监听网卡，改成all就行
inet_interfaces = localhost => inet_interfaces = all
# 第164行的可接收邮件的主机名和域名
mydestination = $myhostname, localhost.$mydomain, localhost => mydestination = $myhostname, $mydomain
# 第448追加一行，转发给dovecot处理
mailbox_command = /usr/libexec/dovecot/deliver -d "$USER" -f "$SENDER" -        a "$RECIPIENT"

```

## 创建邮箱账户

```
# useradd mailreceiver
# passwd 123456789
# id mail # 查看Mail组
# mail 12 12 12
# usermod -g 12 mailreceiver # 修改为mail组成员 
```

## 重启postfix
```
systemctl restart postfix
systemctl enable postfix
```

## 安装dovecot
```
# yum install dovecot
# 如果已经 安装了，需要更新到2.3
# vim /etc/yum.repos.d/dovecot.repo
[dovecot-2.3-latest]
name=Dovecot 2.3 CentOS $releasever - $basearch
baseurl=http://repo.dovecot.org/ce-2.3-latest/centos/$releasever/RPMS/$basearch
gpgkey=https://repo.dovecot.org/DOVECOT-REPO-GPG
gpgcheck=1
enabled=1
# yum makecache
# yum update
```

##  安装pigeonhole 0.5
```
# yum install dovecot-pigeonhole
```
## 配置dovecot

配置dovecot和 sieve
```
# vim /etc/dovecot/dovecot.conf
# 第24行修改协议
protocols = imap pop3 lmtp submission
=> protocols = imap pop3 lmtp sieve # 增加sieve
# 增加
disable_plaintext_auth=no # 关闭文本验证
# 修改
login_trusted_networks = 0.0.0.0/0 # 监听所有登录端口
# 文件最后追加
mail_debug = yes # 打开调式

# vi /etc/dovecot/conf.d/10-mail.conf
# 第25行注释去掉,表示收件箱的位置
mail_location = mbox:~/mail:INBOX=/var/mail/%u
```
## 创建邮件存储目录

```
su - mailreceiver
mkdir -p mail/.imap/INBOX
# 返回root下
exit
```

## 配置Sieve

开启lda的sieve，lda是收，发是lmtp, 如果要处理lmtp也要加上sieve
```
# vim /etc/dovecot/conf.d/15-lda.conf
protocol lda {
  # Space separated list of plugins to load (default is global mail_plugins).
  mail_plugins = $mail_plugins sieve
}

# vim /etc/dovecot/conf.d/20-lmtp.conf
protocol lmtp {
  # Space separated list of plugins to load (default is global mail_plugins).
  mail_plugins = $mail_plugins sieve
}
```
在dovecot 添加plugin
```
# mkdir /var/log/sievelog
# chown mailreceiver.mailreceiver /var/log/sievelog
# chmod 644 /var/log/sievelog

# mkdir /usr/lib/dovecot
# mkdir /usr/lib/dovecot/sieve
# chown mailreceiver.mailreceiver /usr/lib/dovecot/sieve
# chmod 644 /usr/lib/dovecot/sieve

# 文件最后追加
# vim /etc/dovecot/dovecot.conf
plugin {
        sieve_trace_dir = /var/log/sievelog # 日志跟踪
        sieve_trace_level = matching #日志层级
        sieve_trace_debug = yes
        sieve_plugins = sieve_extprograms # 外挂程序
        sieve_pipe_bin_dir = /usr/lib/dovecot/sieve # sieve脚本存放的路径
        sieve_execute_bin_dir = /usr/lib/dovecot/sieve-execute
        sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.environment +vnd.dovecot.execute # 要用到的扩展
        sieve_before = /usr/lib/dovecot/sieve-execute/sieve-execute.sieve
}

```
创建sieve的脚本和shell的脚本
sieve脚本
```
# vim /usr/lib/dovecot/sieve-execute/sieve-execute.sieve
require ["foreverypart", "vnd.dovecot.execute", "copy", "environment", "variables", "body", "mime", "extracttext", "encoded-character"];
if header :matches "Subject" "*"
{
     set "subject" "${1}";
}

foreverypart
{
     if header :mime :type :is "Content-Type" "text"
     {
      extracttext :first 100 "msgcontent";
      break;
     }
}

execute :pipe "report-execute.py" ["${subject}"];

# chmod 755 /usr/lib/dovecot/sieve-execute/sieve-execute.sieve
# chmod +x /usr/lib/dovecot/sieve-execute/sieve-execute.sieve
# chown mailreceiver.mailreceiver /usr/lib/dovecot/sieve-execute/sieve-execute.sieve

```
python脚本
安装mail-parse
```
yum install python-pip
pip install mail-parser
```
```
# vim /usr/lib/dovecot/sieve-execute/report-execute.py
#!/usr/bin/python
import os, sys, mailparser, codecs, re
content = sys.stdin.read()
mail = mailparser.parse_from_string(content)
with open("/tmp/email.txt", "w+") as file:
        file.write(content)
```

重启
```
chmod -R +x /home/mailreceiver/
chmod -R +x /var/mail/mailreceiver
service dovecot restart
tail -1f /var/log/maillog #查看日志
```

错误
通过查看日志，多数都是权限问题，修改文件夹权限为755，文件权限为644
```
tail -1f /var/log/maillog
```

参阅文档
[postfix安装与配置](https://www.cnblogs.com/escwq/p/11869407.html)

[Sieve Pigeonhole配置](https://wiki2.dovecot.org/Pigeonhole/Sieve/Configuration)

[Sieve Pigeonhole安装](https://wiki2.dovecot.org/Pigeonhole/Installation)

[Extprograms Plugins 配置](https://wiki2.dovecot.org/Pigeonhole/Sieve/Plugins/Extprograms)

[可用的插件](https://wiki2.dovecot.org/Pigeonhole/Sieve)

[Sieve 参考例子](https://wiki2.dovecot.org/HowTo/AntispamWithSieve)

[Sieve 语法](http://sieve.info/)

[Pigeonhole sieve例子](https://wiki2.dovecot.org/Pigeonhole/Sieve/Examples)
<!--stackedit_data:
eyJoaXN0b3J5IjpbOTcyMjYyMDIsLTIwMTAwNTc2MzYsNjM5Nz
MzMzQxLDE0ODAwNjk2ODRdfQ==
-->
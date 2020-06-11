


# Postfix Dovecot Sieve
工作流程
 - Postfix 收到邮件传递给dovecot 
 - dovecot启动sieve进行过滤
 - sieve启用sieve extprograms传递给shell
 - shell调用Python
 - python做其他事情

# 系统配置

 - 腾讯云
 - centos 7
 - postfix 2.10.1 
 ```
 #查看版本
 # postconf -d | grep mail_version
 # rpm -qa | grep postfix
 ```
 - dovecot 2.3

## 安装postfix
```
# yum install postfix
```

## 安装dovecot
```
# yum install dovecot
#如果是旧版本的话，更新到2.3
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
protocols = imap pop3 lmtp sieve # 增加sieve
disable_plaintext_auth=no # 关闭文本验证
login_trusted_networks = 0.0.0.0/0 # 监听所有登录端口
mail_debug = yes # 打开调式
```

开启lda的sieve，lda是收，发是lmtp,如果要处理lmtp也要加上sieve
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
# chown mailyi1.mailyi1 /var/log/sievelog
# chmod 755 /var/log/sievelog

# mkdir /usr/lib/dovecot/sieve
# chown mailyi1.mailyi1 /usr/lib/dovecot/sieve
# chmod 755 /usr/lib/dovecot/sieve

# vim /etc/dovecot/dovecot.conf
plugin {
	sieve_trace_dir = /var/log/sievelog # 日志跟踪
	sieve_trace_level = matching # 日志层级
	sieve_plugins = sieve_extprograms # 外挂程序
	sieve_pipe_bin_dir = /usr/lib/dovecot/sieve # sieve脚本存放的路径
	sieve_global_extensions = +vnd.dovecot.pipe # +vnd.dovecot.environment # 要用到的扩展
	sieve_before = /usr/lib/dovecot/sieve/report-spam.sieve # 调用之前执行sieve脚本
}
```
创建sieve的脚本和shell的脚本
sieve脚本
```
# touch /usr/lib/dovecot/sieve/report-spam.sieve
# chmod 755 /usr/lib/dovecot/sieve/report-spam.sieve
# chown mailyi1.mailyi1 /usr/lib/dovecot/sieve/report-spam.sieve
```

shell脚本
```
# touch /usr/lib/dovecot/sieve/sa-learn-spam.sh
# chmod 755 /usr/lib/dovecot/sieve/sa-learn-spam.sh
# chown mailyi1.mailyi1 /usr/lib/dovecot/sieve/sa-learn-spam.sh
```

python脚本
```
```

参阅文档
[postfix安装与配置](https://www.cnblogs.com/escwq/p/11869407.html)
[Sieve Pigeonhole配置](https://wiki2.dovecot.org/Pigeonhole/Sieve/Configuration)
[Sieve Pigeonhole安装](https://wiki2.dovecot.org/Pigeonhole/Installation)
[Extprograms Plugins 配置](https://wiki2.dovecot.org/Pigeonhole/Sieve/Plugins/Extprograms)
[Sieve 参考例子](https://wiki2.dovecot.org/HowTo/AntispamWithSieve)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjk1MDc2MDc4XX0=
-->
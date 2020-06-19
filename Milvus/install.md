## 安装Milvus
安装要求

 - centos 7
 - docker 19.03

## 安装milvus docker镜像
```
# docker pull milvusdb/milvus:0.9.1-cpu-d052920-e04ed5
# mkdir -p /home/$USER/milvus/conf  
# cd /home/$USER/milvus/conf  
# wget https://raw.githubusercontent.com/milvus-io/milvus/v0.9.1/core/conf/demo/server_config.yaml
```

## 启动
本地端口映射给docker
19530 => 19530
19121 => 19121
9091 => 9091
云服务要相应的开启19530,19121,9001端口

本地目录映射给docker目录
/home/root/milvus/db => /var/lib/milvus/db
/home/root/milvus/conf => /var/lib/milvus/conf
/home/root/milvus/logs => /var/lib/milvus/logs
/home/root/milvus/wal => /var/lib/milvus/wal
```
docker run -d --name milvus_cpu_0.9.0 \
-p 19530:19530 \
-p 19121:19121 \
-p 9091:9091 \
-v /home/root/milvus/db:/var/lib/milvus/db \
-v /home/root/milvus/conf:/var/lib/milvus/conf \
-v /home/root/milvus/logs:/var/lib/milvus/logs \
-v /home/root/milvus/wal:/var/lib/milvus/wal \
milvusdb/milvus:0.9.0-cpu-d051520-cb92b1
```

## 确认状态
```
docker ps
```

如果没启动，就看日志
```
# 获得运行 Milvus 的 container ID。
# docker ps -a 
# 检查 docker 日志。
# docker logs <milvus container id>
```

## 安装Milvus Admin

本地端口映射给docker
3000 => 80
```
# docker pull milvusdb/milvus-admin:v0.3.0
# docker run -d -p 3000:80 milvusdb/milvus-admin:v0.3.0
```

浏览器打开ip:3000

## 编译安装
安装要求

 - CentOS 7
 - GCC 7.0 or higher to support C++ 17
 - Git
 - CMake 3.12 or higher

[https://www.cnblogs.com/jixiaohua/p/11732225.html](https://www.cnblogs.com/jixiaohua/p/11732225.html)
```bash
yum -y install git
yum -y install bzip2
yum -y install unzip
cd /usr/local/src

git clone https://github.com/milvus-io/milvus.git # 下载慢可以本地用迅雷下载再传上服务器
cd milvus-master/core
./centos7_build_deps.sh
./build.sh
mkdir /var/lib/milvus
mkdir /var/lib/milvus/bin
cp /usr/local/src/milvus-master/core/cmake_build/src /var/lib/milvus/bin
cp -R /usr/local/src/milvus-master/core/milvus/conf  /var/lib/milvus/
cp -R /usr/local/src/milvus-master/core/milvus/lib /var/lib/milvus/
cp -R /usr/local/src/milvus-master/core/milvus/scripts /var/lib/milvus/


这个可以看情况安装
# cmake安装
wget https://cmake.org/files/v3.12/cmake-3.12.2-Linux-x86_64.tar.gz
tar zxvf cmake-3.12.2-Linux-x86_64.tar.gz
mv cmake-3.12.2-Linux-x86_64 /opt/cmake-3.12.2
ln -sf /opt/cmake-3.12.2/bin/* /usr/bin/


```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMjY3NTQ0MzVdfQ==
-->
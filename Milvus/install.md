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
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI1ODc5NzE1N119
-->
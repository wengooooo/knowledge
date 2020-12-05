安装tensorrt
cuda 10.0
cudnn7.3
```
wget https://developer.download.nvidia.cn/compute/machine-learning/tensorrt/secure/5.0/GA_5.0.2.6/tars/TensorRT-5.0.2.6.Ubuntu-16.04.4.x86_64-gnu.cuda-10.0.cudnn7.3.tar.gz?0lCv3U60fhtg_0Ep9Je__wXebDJE-rSoE-spoKkej693k54p0vxYi1wBq4hclaJMyKR0qb_0gHCoIz2DDZ2l7CAo5PTGINOsu7lH0Y-98g07R7JJ0YHUcYFaVvVx9ydiUXAeeFDGWGRf09OB4neZZfpSD8J-IB3CbY30EJ03UWumRkK24yspc5K33XU-w05z8Ybsia_3MysGJEOlVbL4Dye4aoeSCeUrWWf0VKxXW-pS-OduA5s6leE4Ul_pfzWe

#在home下新建文件夹，命名为tensorrt_tar，然后将下载的压缩文件拷贝进来解压
tar xzvf TensorRT-5.0.2.6.Ubuntu-16.04.4.x86_64-gnu.cuda-9.0.cudnn7.3.tar
 
#解压得到TensorRT-5.0.2.6的文件夹，将里边的lib绝对路径添加到环境变量中
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/root/data/TensorRT-5.0.2.6/lib
 
# 安装TensorRT
cd TensorRT-5.0.2.6/python
pip install tensorrt-5.0.2.6-py2.py3-none-any.whl
 
# 安装UFF,支持tensorflow模型转化
cd TensorRT-5.0.2.6/uff
pip install uff-0.5.5-py2.py3-none-any.whl
 
# 安装graphsurgeon，支持自定义结构
cd TensorRT-5.0.2.6/graphsurgeon
pip install graphsurgeon-0.3.2-py2.py3-none-any.whl
```
安装libnccl2
```
wget https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1604/x86_64/nvidia-machine-learning-repo-ubuntu1604_1.0.0-1_amd64.deb
dpkg -i nvidia-machine-learning-repo-ubuntu1604_1.0.0-1_amd64.deb`
sudo apt-get install -y libnccl2=2.3.7-1+cuda9.0 libnccl-dev=2.3.7-1+cuda9.0
```

安装paddle
```
pip  install  paddlepaddle-gpu
```

测试
```
python

import  paddle.fluid
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkyMDAyODg5N119
-->
## Milvus Vgg Python 使用

 - Milvus 0.9.1
 - Milvus Python Sdk 0.2.12
 - Keras 2.3.1
 - tensorflow             1.14.0

```bash
yum install gcc
yum install openssl-devel -y
yum install zlib-devel -y
mkdir /usr/local/python3
tar -xvf  Python-3.5.6.tgz
cd Python-3.5.6
./configure --prefix=/usr/local/python3
make && make install

ln -s /usr/local/python3/bin/python3.5 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

pip3 install --upgrade pip
```
安装
```bash
pip3 install pymilvus==0.2.12
pip3 install keras==2.3.1
pip3 install tensorflow==1.14
pip3 install image
```

python 代码
```

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4NDYyNDAwNl19
-->
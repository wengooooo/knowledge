# Keras YoloV3

### 安装
```
# pip install keras==2.1.5
# pip install tensorflow-gpu==1.6.0 #如果是gpu版本
# pip install tensorflow==1.6.0 #如果是cpu版本
# pip install Pillow
# pip install matplotlib
```

### 下载复现代码
``` 
# git clone https://github.com/qqwweee/keras-yolo3.git
```

### 创建数据集

```
#在keras-yolo3下创建
VOCdevkit
└── VOC2007
    ├── Annotations
    ├── ImageSets
    │   ├── Layout
    │   ├── Main
    │   └── Segmentation
    ├── JPEGImages
    └── labels
```
在 VOC2007创建文件make_train.py
会Main文件夹生成
test.txt
train.txt
trainval.txt
val.txt
```
import os
import random
 
trainval_percent = 0.1
train_percent = 0.9
xmlfilepath = 'Annotations'
txtsavepath = 'ImageSets/Main'
total_xml = os.listdir(xmlfilepath)
 
num = len(total_xml)
list = range(num)
tv = int(num * trainval_percent)
tr = int(tv * train_percent)
trainval = random.sample(list, tv)
train = random.sample(trainval, tr)
 
if not os.path.exists(txtsavepath):
    print('not exist...{}'.format(txtsavepath))
    os.makedirs(txtsavepath)
 
ftrainval = open('ImageSets/Main/trainval.txt', 'w')
ftest = open('ImageSets/Main/test.txt', 'w')
ftrain = open('ImageSets/Main/train.txt', 'w')
fval = open('ImageSets/Main/val.txt', 'w')
 
for i in list:
    name = total_xml[i][:-4] + '\n'
    if i in trainval:
        ftrainval.write(name)
        if i in train:
            ftest.write(name)
        else:
            fval.write(name)
    else:
        ftrain.write(name)
 
ftrainval.close()
ftrain.close()
fval.close()
ftest.close()
```
### 修改自己需要训练的类别
在YoloV3根目录下找到voc_annotation.py 进行修改
```
# vi voc_annotation.py
classes = ["tops", "dress", "bottoms"]
```

### 下载yolo预训练模型
```
wget https://pjreddie.com/media/files/yolov3.weights
python convert.py yolov3.cfg yolov3.weights model_data/yolo.h5
```

### 修改类别文件
找到model_data/voc_classes.txt修改为自己的类别
```
tops  
dress  
bottoms
```

### 修改anchor文件
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NzgwOTk2M119
-->
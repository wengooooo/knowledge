# YovoV3 Darknet

### 下载源码实现
```
git clone https://github.com/AlexeyAB/darknet.git
```
### 创建数据集目录

```
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
```
test.txt
train.txt
trainval.txt
val.txt
```
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

把VOC的数据集转换为Yolo数据集的格式
在根目录创建voc_to_yolo.py
```
import xml.etree.ElementTree as ET  
import pickle  
import os  
from os import listdir, getcwd  
from os.path import join  
  
sets=[('2007', 'train'), ('2007', 'val'), ('2007', 'test')]  
  
classes = ["tops", "dress", "underwear"]  
  
def convert(size, box):  
  dw = 1./(size[0])  
  dh = 1./(size[1])  
  x = (box[0] + box[1])/2.0 - 1  
  y = (box[2] + box[3])/2.0 - 1  
  w = box[1] - box[0]  
  h = box[3] - box[2]  
  x = x*dw  
  w = w*dw  
  y = y*dh  
  h = h*dh  
  return (x,y,w,h)  
  
def convert_annotation(year, image_id):  
  in_file = open('VOCdevkit/VOC%s/Annotations/%s.xml'%(year, image_id))  
  out_file = open('VOCdevkit/VOC%s/labels/%s.txt'%(year, image_id), 'w')  
  tree=ET.parse(in_file)  
  root = tree.getroot()  
  size = root.find('size')  
  w = int(size.find('width').text)  
  h = int(size.find('height').text)  
  
  for obj in root.iter('object'):  
  difficult = obj.find('difficult').text  
        cls = obj.find('name').text  
        if cls not in classes or int(difficult)==1:  
  continue  
  cls_id = classes.index(cls)  
  xmlbox = obj.find('bndbox')  
  b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text), float(xmlbox.find('ymin').text), float(xmlbox.find('ymax').text))  
  bb = convert((w,h), b)  
  out_file.write(str(cls_id) + " " + " ".join([str(a) for a in bb]) + '\n')  
  
wd = getcwd()  
  
for year, image_set in sets:  
  if not os.path.exists('VOCdevkit/VOC%s/labels/'%(year)):  
  os.makedirs('VOCdevkit/VOC%s/labels/'%(year))  
  image_ids = open('VOCdevkit/VOC%s/ImageSets/Main/%s.txt'%(year, image_set)).read().strip().split()  
  list_file = open('%s.txt'%(image_set), 'w')  
  for image_id in image_ids:  
  list_file.write('%s/VOCdevkit/VOC%s/JPEGImages/%s.jpg\n'%(wd, year, image_id))  
  convert_annotation(year, image_id)  
  list_file.close()
```
运行
```
python voc_to_yolo.py
```

### 修改voc.data和yolov3-voc.cfg文件
```
# vim cfg/voc.data

```shell
classes= 3
train  = %path%/data/voc/train.txt
valid  = %path%/pjreddie/data/voc/2007_test.txt
names = data/voc.names
backup = %path%/backup/
```

### 修改yolov3.cfg
```shell
# vim yolov3.cfg
# 找到
# Testing
batch=1
subdivisions=1
# Training
# batch=64
# subdivisions=16
修改为
# Testing
# batch=1
# subdivisions=1
# Training
batch=4
subdivisions=16

### 搜索yolo, 有三处需要修改，修改这两个参数
filters = 24 #  (classes+5) x 3 公式
classes = 3 # 类别
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU4NDM3NTY1N119
-->
---
title: test
date: 2023-07-06 18:51:30
tags: DeepLearning
categories: DeepLearning
---



Pytorch学习



<!--more-->











# 加载数据

> Dataset 和 DataLoader

* Dataset：获取数据，label，数量
* DataLoader：加载数据，同时可以进行数据预处理





## Dataset

Dataset 是一个抽象类，所有子类都应该**重写 `__getitem__` 方法**； 可以选择性**重写 `__len__`  方法**

* `__getitem__` : 返回数据及其label
* `__len__` : 返回数据数量

### 简单示例

下面是一个简单获取**图像数据集**的示例（label为文件夹的名字，如果label是在txt中，那么获取到名字之后找到相应的 txt 文件拿到 label 即可）

关键代码：

```python
class MyData(Dataset):
    def __init__(self, root_dir, label_dir):    # train  ants    dir_path = "train/ants"  or "train/bees"
        self.root_dir = root_dir
        self.label_dir = label_dir
        self.path = os.path.join(self.root_dir, self.label_dir)  #需要获取文件夹的路径
        self.img_name_list = os.listdir(self.path)  #构造文件名列表    
        
    def __getitem__(self,idx):   #目的：通过index（idx）获取文件，那么就需要创建文件名列表（在__init__中创建）
        img_name = self.img_name_list[idx]   #拿到文件名称
        img_item_path = os.path.join(self.root_dir, self.label_dir, img_name)  #拿到文件路径
        img = Image.open(img_item_path)
        label = self.label_dir
        return img, label
    
    def __len__(self): 
        return len(self.img_path)
```

Jupyter演示：

![image-20230705222407648](./pytorch-learning/image-20230705222407648.png)









## DataLoader







# 训练可视化

有时候需要根据模型的loss来挑选合适的模型，或者查看模型训练结果，使用下面几个方式可以直观的做出可视化

## Tensorboard

Pytorch1.1之后加入了Tensorboard

安装：

```bash
pip install tensorboard
```

pytorch导入包

```python
from torch.utils.tensorboard import SummaryWriter
```



### 读取标量

```python
add_scalar(tag, scalar_value, global_step=None, walltime=None, new_style=False, double_precision=False)
```



```python
writer = SummaryWriter("logs")     #指定文件夹路径

# writer.add_image()  这个方法可以添加图片，下方例子为添加标量数据
# y = x
for i in range(100): 
    writer.add_scalar("y=x",i, i)      #标识名称，竖轴（数据，这里是标量）， 横轴（训练步数）
    
writer.close()
```

然后会在指定位置生成目录（这里是logs）及文件，可以在命令行输入：`tensorboard --logdir=logs`

![image-20230706174413328](./pytorch-learning/image-20230706174413328.png)

也可以指定端口：`tensorboard --logdir=logs --port=6007`

![image-20230706175225682](./pytorch-learning/image-20230706175225682.png)







### 读取图像

```python
add_image(tag, img_tensor, global_step=None, walltime=None, dataformats='CHW')
```

能读入的图像只能是以下几种类型：`img_tensor (torch.Tensor, numpy.ndarray, or string/blobname): Image data`



如何读取numpy型图像，可以利用opencv读取











也直接用numpy读取

导入numpy 

```python
import numpy as np
from PIL import Image
```

```python
img_path = "train/ants_image/0013035.jpg"
img = Image.open(img_path)     #此时是PIL类型的图像数据
img_array = np.array(img)      #转为numpy型数据
```

![image-20230706180848319](./pytorch-learning/image-20230706180848319.png)



使用Tensorboard

```python
writer = SummaryWriter("logs")     #指定文件夹路径

writer.add_image("test_image",img_array,1)
    
writer.close()
```



有时候图片shape不匹配（默认是CHW格式，chanel，height，width）会报错，在使用Tensorboard之前记得看一看图片shape，避免报错，比如我这张图片是HWC格式（通道Chanel在最后）

```python
print(img_array.shape)    #(512, 768, 3)   H W C
```

那么就要指定格式

```python
writer = SummaryWriter("logs")     #指定文件夹路径

writer.add_image("test_image", img_array, 1, dataformats="HWC")
    
writer.close()
```

然后可以在命令行输入`tensorboard --logdir=logs` 然后点击链接查看

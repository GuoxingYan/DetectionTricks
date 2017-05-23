## Install MXNET
1.按照官网指示安装mxnet依赖：[Installing MXNet](http://mxnet.io/get_started/install.html). (选择**Linux-Python-GPU-Build from source**.)  
其中，将`Build the MXNet core shared library`中的`Step 4 Download MXNet sources and build MXNet core shared library.`   
改为： 
       
```
$ git clone --recursive https://github.com/ElaineBao/mxnet.git      
$ cd mxnet/example/rcnn     
$ bash script/additional_deps.sh      
``` 

2.验证安装是否成功：
打开python terminal。  

``` 
$ python
```  
以一个自己定义的symbol为例子查看是否注册成功。  

```
>>> import mxnet as mx
>>> mx.symbol.ROIAlign
```

若出现`<function ROIAlign at xxxx>`注册成功则说明安装成功，否则查看上述步骤是否确已完成。 

## Data Prepare
1.下载imagenet_loc_2017数据集，构造其目录结构为：

```
imagenet_loc_rootpath
|
+---ILSVRC
|      |
|      +--- Annotations
|      |
|      +--- Data
|      |
|      +--- ImageSets
|      |
|      +--- devkit
```

2.清洗数据：   

```
# 生成训练数据集，生成的train.txt文件
python mxnet/example/rcnn/loc_train/clean.py train /the/path/to/ILSVRC2017/ILSVRC

# 生成验证数据集，生成的val.txt文件
python mxnet/example/rcnn/loc_train/clean.py val /the/path/to/ILSVRC2017/ILSVRC
```
把清洗以后生成的`train.txt`重命名成`train_loc.txt`.  

3.完成data prepare后的目录结构为：  

```
imagenet_loc_rootpath
|
+---ILSVRC
|      |
|      +--- Annotations
|      |
|      +--- Data
|      |
|      +--- ImageSets
|      |
|      +--- devkit
+--- train_loc.txt # 清洗以后生成的文件
|
+--- val.txt # 清洗以后生成的文件
```

4.将数据链接到mxnet中，便于使用。 

```
$ cd mxnet
$ cd example/rcnn
$ mkdir data
$ ln -s imagenet_loc_rootpath data/
```

## Model Prepare
1. 将所需要的模型链接到mxnet中，便于使用。模型可以从这里下载: [mxnet model zoo](http://mxnet.io/model_zoo/)

```
$ cd mxnet
$ cd example/rcnn
$ mkdir model
$ ln -s your_model_path model/
```

## Train Imagenet LOC
```
$ cd mxnet
$ cd example/rcnn
$ bash script/resnet_imagenet_loc.sh 0,1  # 0,1表示gpu id
```
关于tricks:  
1. global context: 可以直接在`script/resnet_imagenet_loc.sh`添加`--use_global_context`.  
2. data augmentation: 可以直接在`script/resnet_imagenet_loc.sh`添加`--use_data_augmentation`.  
3. roi align: 可以直接在`script/resnet_imagenet_loc.sh`添加`--use_roi_align`.  
4. multiscale training: 在rcnn/config.py中将  

```
config.SCALES = [(600, 1000)]
```
改成你想要用的multiscale，如：   

```
config.SCALES = [(600, 1000), (800,1333), (1000,1667)]
```

5.constrained positive/negative ratio:在rcnn/config.py中将    

```
# rpn anchors batch size
config.TRAIN.RPN_BATCH_SIZE = 256
# rpn anchors sampling params
config.TRAIN.RPN_FG_FRACTION = 0.5
```
改为：

```
# rpn anchors batch size
config.TRAIN.RPN_BATCH_SIZE = 256
# rpn anchors sampling params
config.TRAIN.RPN_FG_FRACTION = 0.5
```
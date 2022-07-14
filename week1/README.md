## 1.配置linux系统
采用WSL2 Ubuntu安装，然后配置桌面，给ubuntu安装ubuntu-desktop，安装一个X显示服务器程序，把图形界面显示转发到本地的X显示服务器，参考以下方法[WSL2 Ubuntu图形界面使用指南](https://blog.csdn.net/liyunxin_c_language/article/details/114107994)

## 2.Build NCNN
本方法采用的是[how to build NCNN](https://github.com/Tencent/ncnn/wiki/how-to-build#build-for-linux) Build for Linux，在使用Vulkan安装时，先用了Nvida Jetson，发现没有jetson.toolchain, 用以下命令运行Vulkan
```
cd ncnn
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DNCNN_VULKAN=ON -DNCNN_BUILD_EXAMPLES=ON ..
make -j$(nproc)
```
下面来验证一下一些例子
```
../build/examples/squeezenet ../images/256-ncnn.png
[0 llvmpipe (LLVM 12.0.0, 256 bits)]  queueC=0[1]  queueG=0[1]  queueT=0[1]
[0 llvmpipe (LLVM 12.0.0, 256 bits)]  bugsbn1=0  bugbilz=0  bugcopc=0  bugihfa=0
[0 llvmpipe (LLVM 12.0.0, 256 bits)]  fp16-p/s/a=1/1/0  int8-p/s/a=1/1/0
[0 llvmpipe (LLVM 12.0.0, 256 bits)]  subgroup=8  basic=1  vote=1  ballot=1  shuffle=0
WARNING: lavapipe is not a conformant vulkan implementation, testing use only.
532 = 0.163940
920 = 0.093445
716 = 0.061310
```
## 3.量化squeezenet_v1.1 模型
因为我们量化的是squeezenet，可以参考[主要步骤](https://github.com/Tencent/ncnn/blob/master/docs/how-to-use-and-FAQ/quantized-int8-inference.md) 
### (1) 验证一下NCNN推理结果
不使用Vulkan，直接用squeezenet对进行测试，把[测试图片](https://raw.githubusercontent.com/nihui/ncnn-android-squeezenet/master/screenshot.png)
下载到images下,测试squeezenet.
```
$ cd ../examples
$  ../build/examples/squeezenet ../images/screenshot.png
```
测试结果还是比较准确的

<img src="https://raw.githubusercontent.com/nihui/ncnn-android-squeezenet/master/screenshot.png" width="25%" height="25%">

运行squeezenet后的结果

<img src="https://github.com/Shirley866/NCNN-MMdeploy/blob/main/week1/upload_images/b1ffa44b641aea5a81a9a3ed7af5f6a.png">

概率最高的三个类别索引为`281,285，282`，该类别编码+1，可根据[synset_words.txt](https://github.com/Tencent/ncnn/blob/master/examples/synset_words.txt)
得出对应的类别`281 tabby,tabby cat`,`285 Egyptian cat` `282 Tiger cat`，可知结果正确

使用Vulkan
```
cd ncnn
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DNCNN_VULKAN=ON -DNCNN_BUILD_EXAMPLES=ON ..
make -j$(nproc)
```
推理后的结果与vulkan关闭的结果一致，vulkan的推理没有问题
<img src="https://github.com/Shirley866/NCNN-MMdeploy/blob/main/week1/upload_images/dc4307773a4a728ae4ce099dd7d603c.png">
### (2) 量化squeezenet_v1.1 模型
#### a. optimize model
因为这一步会出现一些问题，并且是否optimize并不影响之后的量化结果，所以这一步直接跳过了，直接开始create the calibration table file

#### b. Create the calibration table file
先下载[校准集](https://github.com/nihui/imagenet-sample-images)，把所有图片下载到images文件夹下，

```
$ find images/ -type f > imagelist.txt
$ cd build/tools/quantize/ncnn2table imagelist.txt squeezenet_v1.1.table mean=[104,117,123] norm=[1,1,1]  shape=[227,227,3]  pixel=BGR thread=1  method=kl
```

#### c. Quantize Model

```
$ cd build/tools/quantize/ncnn2int8 squeezenet_v1.1.param squeezenet_v1.1.bin squeezenet_v1.1-int8.param squeezenet_v1.1-int8.bin
```

#### 4.Inference

因为在推理的时候，并没有重新编译一个int8模型，所以直接把squeezenet_v1.1-int8.param squeezenet_v1.1-int8.bin 放在int8 文件夹中，并且把名字改为squeezenet_v1.1.param squeezenet_v1.1.bin

```
$cd int8
$../build/examples/squeezenet screenshot.png
```
最后推理结果证明是正确的,但是精度有所下降。

<img src="https://github.com/Shirley866/NCNN-MMdeploy/blob/main/week1/upload_images/8cebf8062f617ca1199b2dc720f8ca3.png">

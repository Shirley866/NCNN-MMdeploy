## 1.配置linux系统
采用WSL2 Ubuntu安装，然后配置桌面，给ubuntu安装ubuntu-desktop，安装一个X显示服务器程序，把图形界面显示转发到本地的X显示服务器，参考以下方法[WSL2 Ubuntu图形界面使用指南](https://blog.csdn.net/liyunxin_c_language/article/details/114107994)

## 2.Build NCNN
本方法采用的是[how to build NCNN](https://github.com/Tencent/ncnn/wiki/how-to-build#build-for-linux) Build for Linux，在使用Vulkan安装时，先用了Nvida Jetson，发现没有jetson.toolchain, 用以下命令运行Vulkan
```
cd ncnn
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../toolchains/jetson.toolchain.cmake -DNCNN_VULKAN=ON -DNCNN_BUILD_EXAMPLES=ON ..
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
### (1) Build NCNN
不使用Vulkan，直接用squeezenet对进行测试，把[测试图片](https://raw.githubusercontent.com/nihui/ncnn-android-squeezenet/master/screenshot.png)
下载到images下,测试squeezenet.
```
$ cd ../examples
$  ../build/examples/squeezenet ../images/screenshot.png
```
测试结果还是比较准确的
![alt 测试图片](https://raw.githubusercontent.com/nihui/ncnn-android-squeezenet/master/screenshot.png")

## 1.配置linux系统
采用WSL2 Ubuntu安装，然后配置桌面，给ubuntu安装ubuntu-desktop，安装一个X显示服务器程序，把图形界面显示转发到本地的X显示服务器，参考以下方法[WSL2 Ubuntu图形界面使用指南](https://blog.csdn.net/liyunxin_c_language/article/details/114107994)

## 2.Build NCNN
本方法采用的是[how to build NCNN](https://github.com/Tencent/ncnn/wiki/how-to-build#build-for-linux) Build for Linux，在使用Vulkan安装时，先用了Nvida Jetson，发现没有jetson.toolchain,用以下命令打开Vulkan
```
cd ncnn
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../toolchains/jetson.toolchain.cmake -DNCNN_VULKAN=ON -DNCNN_BUILD_EXAMPLES=ON ..
make -j$(nproc)
```

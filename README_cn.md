# Foundationpose-CPP
## Windows
首先感谢原作，由于我的常用部署环境是windows，所以为了在windows 10/11 上运行，修改了[原repo](https://github.com/zz990099/foundationpose_cpp.git)
推理环境: CUDA11.8 + CUDNN8.9 + TensorRT8.6.1.6
### 1. 基础依赖
为了方便，这里使用的是`vcpkg`。具体使用方法可以参考[官方文档](https://learn.microsoft.com/zh-cn/vcpkg/)
```shell
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.bat
./vcpkg.exe install assimp opencv4 eigen3 gtest glog 
```
### 2. CV-CUDA
目前[CV-CUDA](https://github.com/CVCUDA/CV-CUDA.git)目前(20250730)尚不支持windows平台，所以本分支的主要修改在于如何用ppl.cv替换CV-CUDA, 下面编译ppl.cv
```shell
git clone https://github.com/openppl/ppl.cv.git
cd ppl.cv
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../install ^
-DPPLCV_USE_X86_64=ON -DPYBIND11_NOPYTHON=ON ^
-DPPLCV_USE_CUDA=ON -DPPLCV_INSTALL=ON -DPPLCOMMON_INSTALL=ON
cmake --build . --config Release --target ALL_BUILD
cmake --build . --config Release --target install
```
### 3. 编译本仓库
```shell
git clone https://github.com/wawatt/foundationpose_cpp_windows.git
cd foundationpose_cpp
git submodule init
git submodule update
```
- 去除MSVC不支持的-Wextra -Wdeprecated 选项
[easy_deploy_tool\deploy_core](easy_deploy_tool\deploy_core\CMakeLists.txt)和
[easy_deploy_tool\inference_core\trt_core](easy_deploy_tool\inference_core\trt_core\CMakeLists.txt) 
```shell
cd foundationpose_cpp
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=D:/build/vcpkg/scripts/buildsystems/vcpkg.cmake ^
-DENABLE_TENSORRT=ON -DTRT_DIR=E:/ENVs/TensorRT-8.6.1.6 ^
-Dpplcv_DIR="E:/ENVs/ppl.cv/install/lib/cmake/ppl" ^
-DCMAKE_CUDA_ARCHITECTURES="86;89"
cmake --build . --config Release --target ALL_BUILD
```
### 4. onnx转trt模型
[脚本](tools\win.md)
### 5. 运行
```shell
cd foundationpose_cpp_windows
"build/bin/Release/simple_tests.exe" --gtest_filter=foundationpose_test.test
```
以下文本未作修改...

## About this project

该项目基于[nvidia-issac-pose-estimation](https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_pose_estimation)改写，去除了原项目中的复杂依赖，能够使用`FoundationPose`的Python工程[FoundationPose](https://github.com/NVlabs/FoundationPose)导出的onnx模型来做推理，部署应用十分方便。

**Notes:** 该项目只包含了`FoundationPose`部分的代码，实际上6D位姿检测的运行，还依赖于目标物的掩码，需要运行类似`SAM`的算法，[EasyDeploy](https://github.com/zz990099/EasyDeploy)项目下提供了`MobileSAM`和`NanoSAM`的算法实现和推理优化，可供参考。

## Update LOG

- [2025.04] 解耦`Register`和`Track`过程; 令算法输出mesh下的位姿，提供mesh_loader相关的接口和方法供外部拓展。[对应PR](https://github.com/zz990099/foundationpose_cpp/pull/21)
- [2025.03] 渲染过程与原Python工程对齐，支持无texture纹理输入渲染. [对应PR](https://github.com/zz990099/foundationpose_cpp/pull/13).
- [2025.03] 添加对Jetson Orin平台支持，[一键配置docker环境](docs/build_enviroment_on_jetson.md)

## Features

1. 去除了原工程的复杂环境构建过程，以及各种依赖项问题，能够轻松适配到其他项目工程中。
2. 对`FoundationPose`算法本身做了封装，**支持动态尺寸图像输入**，简单灵活。
3. 提供了基于`BundleSDF`生成目标物三维模型的[脚本教程](./docs/gen_3d_obj_with_bundlesdf.md)。
4. :fire: 支持Orin开发板(Orin-NX-16GB)

## Demo

运行公开数据`mustard`模型检测结果:

| <img src="./assets/test_foundationpose_register.png" alt="1" width="500"> |
|:----------------------------------------:|
| **foundationpose(fp16) Register test result**  |

| <img src="./assets/test_foundationpose_track.gif" alt="1" width="500"> |
|:----------------------------------------:|
| **foundationpose(fp16) Track test result**  |

以下是在`nvidia-4060-8G`, `i5-12600kf`硬件上执行结果

|  nvidia-4060-8G   |   fps   |  cpu   |   gpu   |
|:---------:|:---------:|:----------------:|:----------------:|
|  foundationpose(fp16)-Register   |   2.8   |  100%   |  6.5GB   |
|  foundationpose(fp16)-Track   |   220   |  100%   |  5.8GB   |

以下是在`jetson-orin-nx-16GB`硬件上执行结果

|  jetson-orin-nx-16GB   |   fps   |  cpu   |   mem_total   |
|:---------:|:---------:|:----------------:|:----------------:|
|  foundationpose(fp16)-Register   |   0.6   |  15%   |  5.6GB(5.5GB on gpu)   |
|  foundationpose(fp16)-Track   |   100   |  60%   |  5.1GB(5.0GB on gpu)   |


## Usage

### Enviroment Build

1. 下载`foundationpose_cpp`repo
```bash
git clone git@github.com:zz990099/foundationpose_cpp.git
cd foundationpose_cpp
git submodule init
git submodule update
```

2. 使用`docker`来构建运行环境
  ```bash
    cd ${foundationpose_cpp}
    bash easy_deploy_tool/docker/easy_deploy_startup.sh
    # choose `jetson` -> `trt10_u2204`/`trt8_u2204` (`trt8_u2004` will not work)
    bash easy_deploy_tool/docker/into_docker.sh
  ```

### Convert Models

1. 从[google drive](https://drive.google.com/drive/folders/1AmBopDz-RrykSZVCroDH6jFc1-k8HkL0?usp=drive_link)中下载onnx模型文件，放到`/workspace/models/`文件夹下。

2. 运行模型转换脚本
  ```bash
    cd /workspace
    bash tools/cvt_onnx2trt.bash
  ```

### Compile Code

1. 编译整个工程
  ```bash
    cd /workspace
    mkdir build && cd build
    cmake -DENABLE_TENSORRT=ON ..
    make -j
  ```

### Run demo

#### 运行公开数据集demo ---- mustard

1. 下载数据集，放到`/workspace/test_data/`下，并解压，[下载地址](https://drive.google.com/drive/folders/1pRyFmxYXmAnpku7nGRioZaKrVJtIsroP)

2. 直接运行测试用例即可
  ```bash
    cd /workspace/build
    ./bin/simple_tests --gtest_filter=foundationpose_test.test
  ```

#### 自制三维模型

1. 参考[利用BundleSDF生成三维模型](./docs/gen_3d_obj_with_bundlesdf.md)

2. 根据您的自定义数据，修改`/workspace/simple_tests/src/test_foundationpose.cpp`下的路径，重新编译。

3. 运行测试用例
  ```bash
    cd /workspace/build
    ./bin/simple_tests --gtest_filter=foundationpose_test.test
  ```

在`/workspace/test_data/`下，可以看到`Register`和`Track`两个过程的结果。

## References

- [nvidia-isaac-pose-estimation](https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_pose_estimation)
- [FoundationPose](https://github.com/NVlabs/FoundationPose)
- [BundleSDF](https://github.com/NVlabs/BundleSDF)
- [XMem](https://github.com/hkchengrex/XMem)
- [EasyDeploy](https://github.com/zz990099/EasyDeploy)

有任何问题，欢迎联系`771647586@qq.com`

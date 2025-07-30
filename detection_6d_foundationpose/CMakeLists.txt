cmake_minimum_required(VERSION 3.8)
project(detection_6d_foundationpose)

add_compile_options(-std=c++17)
add_compile_options(-O3)
set(CMAKE_CXX_STANDARD 17)

set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)


# packages dep
set(pplcv_DIR "" CACHE PATH "Path to ppl.cv")
find_package(pplcv REQUIRED)
find_package(glog REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(assimp REQUIRED)

# cuda dep
find_package(CUDA REQUIRED)
find_package(CUDAToolkit REQUIRED)
# find_package(nvcv_types REQUIRED)
# find_package(cvcuda REQUIRED)
# set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
enable_language(CUDA)


set(source_file src/nvdiffrast/common/cudaraster/impl/Buffer.cpp
                src/nvdiffrast/common/cudaraster/impl/CudaRaster.cpp
                src/nvdiffrast/common/cudaraster/impl/RasterImpl.cpp
                src/nvdiffrast/common/cudaraster/impl/RasterImpl.cu
                src/nvdiffrast/common/common.cpp
                src/nvdiffrast/common/interpolate.cu
                src/nvdiffrast/common/rasterize.cu
                src/nvdiffrast/common/texture.cu
                src/foundationpose_render.cpp
                src/foundationpose_render.cu
                src/foundationpose_sampling.cpp
                src/foundationpose_sampling.cu
                src/foundationpose_utils.cu
                src/foundationpose_decoder.cu
                src/foundationpose.cpp
                src/mesh_loader/assimp_mesh_loader.cpp
)

include_directories(
  include
  ${OpenCV_INCLUDE_DIRS}
  ${CUDA_INCLUDE_DIRS}
  ${PPLCV_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED ${source_file})


target_link_libraries(${PROJECT_NAME} PUBLIC
  Eigen3::Eigen
  ${OpenCV_LIBS}
  glog::glog
  deploy_core
  trt_core
  assimp::assimp
  # nvcv_types
  # cvcuda
  ${PPLCV_LIBRARIES}
)


target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)

set(TRT_DLLS 
  ${TRT_DIR}/lib/nvinfer.dll
  ${TRT_DIR}/lib/nvinfer_plugin.dll
)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    ${TRT_DLLS}
    $<TARGET_FILE_DIR:${PROJECT_NAME}>)
cmake_minimum_required(VERSION 3.8)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
project(foundationpose_cpp)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
# 处理easy_deploy_tool中的u_char, 尽量避免代码修改
add_compile_definitions(u_char=uint8_t)

set(TRT_DIR "" CACHE PATH "Path to TensorRT")
include_directories(${TRT_DIR}/include)
link_directories(${TRT_DIR}/lib)

add_subdirectory(easy_deploy_tool)
add_subdirectory(detection_6d_foundationpose)

add_subdirectory(simple_tests)

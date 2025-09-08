cmake_minimum_required(VERSION 3.10)
project(InferenceLibrary LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# CUDA path from env (should be valid on both Win & Linux)
set(CUDA_PATH "$ENV{CUDA_PATH}")

# --- PLATFORM-SPECIFIC paths ---
if(WIN32)
    set(TENSORRT_PATH "C:/TensorRT-10.5.0.18")
    set(CUDA_LIB_PATH "${CUDA_PATH}/lib/x64")
else()
    set(TENSORRT_PATH "/usr/local/tensorrt-10.5")
    set(CUDA_LIB_PATH "${CUDA_PATH}/lib64")
endif()

# Build configuration option
option(STANDALONE_TEST "Build standalone test executable" OFF)

# Add targets based on configuration
if(STANDALONE_TEST)
    add_executable(inference "./inference_main.cpp")
    target_compile_definitions(inference PRIVATE STANDALONE_TEST=1)
else()
    add_library(inference SHARED "./inference_main.cpp")
endif()

# Include directories
target_include_directories(inference PRIVATE
    "${CUDA_PATH}/include"
    "${TENSORRT_PATH}/include"
)

# Library directories
link_directories(
    "${CUDA_LIB_PATH}"
    "${TENSORRT_PATH}/lib"
)

# Link libraries
if(WIN32)
    target_link_libraries(inference
        "${TENSORRT_PATH}/lib/nvinfer_10.lib"
        "${TENSORRT_PATH}/lib/nvonnxparser_10.lib"
        "${CUDA_LIB_PATH}/cudart.lib"
    )
    # Windows requires the __stdcall calling convention for JNI. "/Gz" is the flag for this.
    target_compile_options(inference PRIVATE "/Gz")
else()
    target_link_libraries(inference
        "${TENSORRT_PATH}/lib/libnvinfer.so"
        "${TENSORRT_PATH}/lib/libnvonnxparser.so"
        "${CUDA_LIB_PATH}/libcudart.so"
    )
endif()

# Position-independent code
set_target_properties(inference PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Ignore deprecated warnings from TensorRT
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(inference PRIVATE "-Wno-deprecated-declarations")
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(inference PRIVATE "/wd4996")
endif()

# TensorRT Installation Guide for Linux
3-26-2025
9-07-2025: Update for Linux aarch64
10-15-2025: Rearrange for README.md


This guide covers TensorRT installation for both Linux x86_64 and Linux aarch64 platforms.

## Prerequisites

You'll need to create an NVIDIA Developer account to download TensorRT packages.

## Installation Steps

### For Linux x86_64

1. Go to the TensorRT download page:
   https://developer.nvidia.com/tensorrt/download/10x

2. Download the package "TensorRT 10.5 GA for Linux x86_64 and CUDA 12.0 to 12.6 TAR Package"

3. Extract the downloaded `.tar` file:
   ```shell
   tar -xzvf TensorRT-10.5.0.18.linux.x86_64-gnu.cuda-12.6.tar.gz
   ```

4. Move the TensorRT folder to `/usr/local`:
   ```shell
   sudo mv TensorRT-10.5.0.18 /usr/local/tensorrt-10.5
   ```

### For Linux aarch64

1. Go to the TensorRT download page:
   https://developer.nvidia.com/tensorrt/download/10x

2. Navigate to the "TensorRT 10.5 GA for JetPack" section and download "TensorRT 10.5 GA for L4T and CUDA 12.6 TAR Package"

3. Extract the downloaded `.tar` file:
   ```shell
   tar -xzvf TensorRT-10.5.0.18.l4t.aarch64-gnu.cuda-12.6.tar.gz
   ```

4. Move the TensorRT folder to `/usr/local`:
   ```shell
   sudo mv TensorRT-10.5.0.18 /usr/local/tensorrt-10.5
   ```

## Environment Setup

Define env var (as Windows installer does) by adding the 
following to your `~/.bashrc`

```shell
echo "export CUDA_PATH=/usr/local/cuda-12.6:${CUDA_PATH}" >> ~/.bashrc
echo "export CUDA_PATH=/usr/local/tensorrt-10.5/bin:${CUDA_PATH}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/tensorrt-10.5/lib:${LD_LIBRARY_PATH}" >> ~/.bashrc
echo "export CPATH=/usr/local/tensorrt-10.5/include:${CPATH}" >> ~/.bashrc
source ~/.bashrc
```

## Check the installation
Verify your installation by running these commands:
```shell
make --version
java --version
nvidia-smi
nvcc --version
dpkg -l | grep nvinfer
```

## Notes

- Both x86_64 and aarch64 installations follow the same process, only the download package differs
- Make sure CUDA 12.6 is installed before proceeding with TensorRT installation
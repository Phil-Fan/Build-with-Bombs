3-26-2025

9-07-2025: Update for Linux aarch64

## Steps

1. Go to the following page (NVIDIA forces you to make an account in order to download):
https://developer.nvidia.com/tensorrt/download/10x

2. Download the following `.tar` package:(The `.deb` package didn't work for me because it was missing the ONNX parser.)

* For Linux x86_64, install the package "TensorRT 10.5 GA for Linux x86_64 and CUDA 12.0 to 12.6 TAR Package". 
* For Linux aarch64, go to the "TensorRT 10.5 GA for JetPack" section and install the package "TensorRT 10.5 GA for L4T and CUDA 12.6 TAR Package". 
* Extract the `.tar` and move the contents to `/usr/local`

```
tar -xzvf TensorRT-10.5.0.18.l4t.aarch64-gnu.cuda-12.6.tar.gz
```

You will see a `TensorRT-10.5.0.18` folder in the current directory.


3. Move the TensorRT folder to `/usr/local`

```shell 
sudo mv TensorRT-10.5.0.18 /usr/local/tensorrt-10.5
```


4. Define env var (as Windows installer does) by adding the following to your `~/.bashrc`

```shell 
echo "export CUDA_PATH=/usr/local/cuda-12.6:${CUDA_PATH}" >> ~/.bashrc
echo "export CUDA_PATH=/usr/local/tensorrt-10.5/bin:${CUDA_PATH}" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/tensorrt-10.5/lib:${LD_LIBRARY_PATH}" >> ~/.bashrc
echo "export CPATH=/usr/local/tensorrt-10.5/include:${CPATH}" >> ~/.bashrc
source ~/.bashrc
```

## Check the installation

```shell 
make --version
java --version
nvidia-smi
nvcc --version
dpkg -l | grep nvinfer
```

## Build and Test Options

You can choose to build either the shared library (inference.so) for production use, or a standalone test executable for testing the installation.

### Option 1: Using Make (Recommended)

```shell
# Build the shared library
make lib

# Build the test executable
make test

# Build both library and test executable
make all

# Run the test
make run

# Clean all build files
make clean
```

### Option 2: Using CMake Directly (Alternative Method)

If the Makefile is not available or you need more control over the build process, you can use CMake directly:

1. Build the shared library (inference.so)
    ```shell
    # Create a build directory for the shared library
    mkdir build_lib
    cd build_lib
    cmake ..
    cmake --build . --config Release
    ```
    This will generate `libinference.so` in the build_lib directory.

2. Build the test executable
    ```shell
    # Create a build directory for the test executable
    mkdir build_test
    cd build_test
    # This will automatically enable test mode without manual code changes
    cmake .. -DSTANDALONE_TEST=ON
    cmake --build . --config StandaloneTest
    ```
    This will generate the `inference` executable in the build_test directory. When building with STANDALONE_TEST=ON:
    - Test code is automatically enabled
    - Console output is enabled (no log file redirection)
    - No manual code changes are needed

Note: Both methods will produce the same output files. Choose the method that best suits your needs.

4. run the inference

    ```shell
    ./inference
    ```

    it will build two parallel diffusion threads, and for each thread, it will diffuse 1000 timesteps.

    you will see the output in the console like:

    ```
    job: 0, step = X, sum = Y
    job: 1, step = X, sum = Y
    ```

    If you can see the `step` goes to 0, and the `sum` is not 0, then the installation of TensorRT is successful.
    
    
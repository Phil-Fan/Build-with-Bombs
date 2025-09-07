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

### tiny demo to check the installation

1. change line 831 in [inference_main.cpp](inference_main.cpp#L831) into

    ```cpp
    #if 1
    ```
    comment out `freopen` on line 334 [inference_main.cpp](inference_main.cpp#L334) (this allows printfs to work and not go to the .log")

    ```cpp
    // freopen("buildwithbombs.log", "w", stdout);
    ```
2. change line 20 in [CMakeLists.txt](CMakeLists.txt#L20) into

    ```cmake
    add_executable(inference "./inference_main.cpp")
    ```
    
3. build the inference DLL using CMake.

    ```shell
    mkdir build
    cd build
    cmake ..
    cmake --build . --config Release
    ```

    it will generate a executable file `inference` in the `build` folder.

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
    
    
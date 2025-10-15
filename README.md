# Build with Bombs 💣
## A Minecraft Java mod for procedural house generation by diffusion

This repo contains two components:

- `inference_dll` This contains a C++ DLL that sets up CUDA and calls TensorRT. It provides a number of exported functions for use by Java.
- `mod_neoforge` This is the Java mod code that calls the inference.dll functions. It handles getting / setting blocks in Minecraft.

Version requirements:
- neoforge-21.1.77
- Minecraft-1.21.1
- TensorRT-10.5.0.18
- CUDA 12.6

## Hardware compatibility
TensorRT 10.5 requires an NVIDIA GPU with compute capability >= 7.5. This means it requires an **RTX 2060** or better, a **GTX 1660 Ti** or better, an **MX550** or better, or a **Tesla T4** or better. See the [support matrix](https://docs.nvidia.com/deeplearning/tensorrt/archives/tensorrt-1050/support-matrix/index.html). Check this Wikipedia table to find the compute capability of your GPU: [Compute capability, GPU semiconductors and Nvidia GPU board products](https://en.wikipedia.org/wiki/CUDA#GPUs_supported)

## Setup Guide
This setup guide includes steps for building the `.jar` Java mod file as well as building the native executable.

### Required packages and programs

* CMake: https://cmake.org/download/
* Java 21 JDK: https://www.oracle.com/java/technologies/downloads/#jdk21-windows
* CUDA 12.6: https://developer.nvidia.com/cuda-12-6-0-download-archive
* TensorRT 10.5: https://developer.nvidia.com/tensorrt/download/10x
* (Optional) Make: https://www.gnu.org/software/make/

You can check your environment configuration by running the following commands:

```shell
cmake --version

java --version

nvidia-smi
nvcc --version

dpkg -l | grep nvinfer

make --version
```

Check [tensorrt_linux_install_steps.md](inference_dll/tensorrt_linux_install_steps.md) for TensorRT installation details.


### Build steps (Windows)

1. Run `./gradlew build`. After the build succeeds, the mod .jar file will be located at `mod_neoforge/build/libs/buildwithbombs-0.2.1.jar`.

2. Build the inference library and test executable using CMake:
   ```shell
   cd inference_dll
   mkdir build
   cd build
   cmake ..
   cmake --build . --config Release
   ```
   This will produce `inference.dll` in the `inference_dll/build/Release` directory.

3. Create a `run` directory inside `mod_neoforge` if it does not exist, and copy the generated DLL

4. Download the ONNX model from the [release page](https://github.com/timothy-barnes-2357/Build-with-Bombs/releases/download/v0.2.1/ddim_single_update.onnx) and place it in the `mod_neoforge/run` directory. This contains the model 
parameters and must be located next to `inference.dll`.

5. Make sure `inference.dll` is able to find the TensorRT and CUDA dynamic libraries. Either copy all DLLs into the `mod_neoforge/run` directory, or add the CUDA and TensorRT lib folders to the system path.

6. Test the mod by running:
   ```
   ./gradlew runClient
   ```

### Build steps (Linux x86_64)

1. Run `./gradlew build`. After the build succeeds, the mod .jar file will be located at `mod_neoforge/build/libs/buildwithbombs-0.2.1.jar`.

2. Build the inference library and test executable:

   Option 1: Using Make (Recommended)
   ```shell
   cd inference_dll
   
   # Build the shared library
   make lib
   ```

   Option 2: Using CMake directly
   ```shell
   cd inference_dll
   mkdir build
   cd build
   cmake ..
   cmake --build . --config Release
   ```

3. Copy the newly built library (`libinference.so`) to the mod's run folder. Create the `run` folder if it doesn't exist:
   ```shell
   cp libinference.so ../mod_neoforge/run
   ```

4. Download the ONNX model from the [release page](https://github.com/timothy-barnes-2357/Build-with-Bombs/releases/download/v0.2.1/ddim_single_update.onnx) and place it in the `mod_neoforge/run` directory. This contains the model 
parameters and must be located next to `libinference.so`.

5. Make sure `libinference.so` can find the TensorRT and CUDA shared libraries. Either copy all required `.so` files into `mod_neoforge/run`, or add the TensorRT and CUDA library folders to your system path:
   ```shell
   export LD_LIBRARY_PATH=/usr/local/tensorrt-10.5/lib:$LD_LIBRARY_PATH
   ```

6. Test the mod by running:
   ```shell
   ./gradlew runClient
   ```


### Build steps (Linux aarch64)

> There are still some issues with the Linux aarch64 build. Before we fix them, you can try the following steps to build the mod.

1. following the steps in [tensorrt_linux_install_steps.md](inference_dll/tensorrt_linux_install_steps.md) to install TensorRT and CUDA.

2. Locate the following CUDA and TensorRT and copy them to your Minecraft run directory (the parent directory of /mods).
    (usually in `/usr/local/tensorrt-10.5/lib`)
    * libnvinfer_builder_resource.so.10.5.0
    * libnvinfer.so.10.5.0

    (usually in `/usr/local/cuda-12.6/lib64`)
    * libnvonnxparser.so
    * libcudart.so.12

3. Place `buildwithbombs-0.2.1.jar`(build from previous step) in the Minecraft `/mods` folder
4. Place `libinference.so` (build from previous step) in the Minecraft run directory.
5. Place ddim_single_update.onnx (from [release page](https://github.com/timothy-barnes-2357/Build-with-Bombs/releases/tag/v0.2.1)) in the Minecraft run directory.
6. Start the game. If it loads, you will be given "Diffusion TNT" items upon entering a world. Placing one of these blocks triggers the diffusion process.

once you move all the files to the run directory, your folder structure should be like this:

```
.
├── ddim_single_update.onnx
├── libcudart.so.12
├── libinference.so
├── libnvinfer_builder_resource.so.10.5.0
├── libnvinfer.so.10
├── libnvonnxparser.so
├── ···
├── mods
│   └── buildwithbombs-0.2.1.jar
└── versions
    └── 1.21.1-NeoForge
```

6. Start the game. If it loads, you will be given "Diffusion TNT" items upon entering a world. Placing one of these blocks triggers the diffusion process.





### Building and running the standalone test

To test your TensorRT installation without running Minecraft, you can build and run a standalone test executable:


**On Linux:**

```shell
cd inference_dll

# Build the shared library libinference.so
make lib

# Build the test executable inference
make test 

# Run the test
make run

# Clean all build files
make clean
```

**On Windows:**


1. Build the shared library (inference.dll)
    ```shell
    # Create a build directory for the shared library
    mkdir build_lib
    cd build_lib
    cmake ..
    cmake --build . --config Release
    ```
    This will generate `inference.dll` in the build_lib directory.

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

3. Run the inference

    ```shell
    ./inference.exe
    ```


the test will build two parallel diffusion threads, and for each thread, it will diffuse 1000 timesteps.

In the end, you will see the output in the console like:

```
job: 0, step = X, sum = Y
job: 1, step = X, sum = Y
```

If you can see the `step` goes to 0, and the `sum` is not 0, then the installation of TensorRT is successful.


### Problem you might encounter

```
ERROR: Mod and diffusion engine don't match! Init failed.
```

That means you have different version of the mod and the diffusion engine. you should update your code by `git pull` and build `buildwithbombs-0.2.1.jar` again.




## Social

[buildwithbombs.com](https://buildwithbombs.com)

[Discord to ask questions](https://discord.gg/2ym2tUV5E3)

Join this server to try it out (no client-side mod required): `mc.buildwithbombs.com` 🧨

# Docker + OpenFOAM&reg; + PyTorch

## Overview

The Dockerfile in this repository creates an image with [OpenFOAM-plus](https://openfoam.com/) and [PyTorch](https://pytorch.org/) support. The image is currently based on
- Ubuntu 18.04,
- OpenFOAM-v1912, and
- PyTorch 1.4 (without GPU support).

## How to build the image

To build the image yourself, copy this repository and navigate into the folder:
```
git clone https://github.com/AndreWeiner/of_pytorch_docker.git
cd of_pytorch_docker
```
If you want to upload the image to a Docker registry, consider the following naming convention when running the build command:
```
docker build -t your_registry_name/of_pytorch:of1912-py1.4-cpu -f Dockerfile.abi .
```

## Dockerfile details and practical tips

### PyTorch versions 1.3 and later (with ABI)

Since version 1.3 of PyTorch, there is a version of libtorch compiled with ABI enabled ([read more](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html)). However, it is still necessary to build the OpenFOAM sources because the compiler version in the default centOS-based docker image uses a very old compiler (gcc 4.8) which is not compatible with current versions of PyTorch.

### Prior to PyTorch 1.3 (no ABI)

OpenFOAM is installed system-wide in the */opt/OpenFOAM* folder. The only change compared to a regular compilation is the additional flag
```
-D_GLIBCXX_USE_CXX11_ABI=0
```
which is necessary to build OpenFOAM solvers and utilities including PyTorch code/libraries with the GNU compiler (the flag is related to backwards compatibility for standards older than C++11, [read more](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html)).

The previous Dockerfile has been renamed to *Dockerfile.no_abi* (OpenFOAM-v1906 and PyTorch 1.2).

### Libtorch

The PyTorch library files are located in */opt/libtorch*. The environment variable **TORCH_LIBRARIES** can be used to indicate the location of certain header and library files to the compiler and linker. To compile an OpenFOAM app with PyTorch, add
```
EXE_INC = \
... \
-I$(TORCH_LIBRARIES)/include \
-I$(TORCH_LIBRARIES)/include/torch/csrc/api/include
```
to the include paths, and
```
EXE_LIBS = \
...\
-rdynamic \
-Wl,-rpath,$(TORCH_LIBRARIES)/lib $(TORCH_LIBRARIES)/lib/libtorch.so $(TORCH_LIBRARIES)/lib/libc10.so \
-Wl,--no-as-needed,$(TORCH_LIBRARIES)/lib/libcaffe2.so \
-Wl,--as-needed $(TORCH_LIBRARIES)/lib/libc10.so \
-lpthread
```
to the library paths and linker flags.

# opencv4.5.4-cuda11.8-cudnn-ubuntu-install
Installation guide for opencv 4.5.4 with cuda 11.8 and cudnn on ubuntu

Here is a small guide to help you install **opencv 4.5.4** with **cuda 11.8** and **cudnn** for ubuntu linux. I made this guide because I had a lot of trouble at the beginning between the different versions of opencv and cuda. Now, I know how to do it and so I decided to share this with you.

This guide also works for other versions of opencv 4.5.x with cuda 11.8. 

**Beware, there are some compatibility issues between opencv 4.6 and cuda 12.x at this time**.

## 0 - Define env. variables

Depending on your ubuntu version, the commands may differ slightly. I advise you to define an environment variable as follows:

ubuntu 18.04:
```bash 
export OS=ubuntu1804
```


ubuntu 20.04:
```bash 
export OS=ubuntu2004
```


ubuntu 22.04:
```bash 
export OS=ubuntu2204
```

cudnn version
```bash 
export cudnn_version=8.8.1.*
```

cuda version
```bash 
export cuda_version=cuda11.8
```

cuda architecture
```bash 
##depending on your gpu compute cuda capability, for me it is 7.5
#see https://developer.nvidia.com/cuda-gpus
export cuda_arch=7.5
```

## 1 - Install tools and required libraries

```bash

$ sudo apt update

$ sudo apt upgrade

$ sudo apt install build-essential cmake pkg-config unzip yasm git checkinstall

$ sudo apt install libjpeg-dev libpng-dev libtiff-dev

$ sudo apt install libavcodec-dev libavformat-dev libswscale-dev libavresample-dev

$ sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev

$ sudo apt install libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev 

$ sudo apt install libfaac-dev libmp3lame-dev libvorbis-dev

$ sudo apt install libopencore-amrnb-dev libopencore-amrwb-dev

$ sudo apt-get install python3-dev python3-pip

$ sudo -H pip3 install -U pip numpy

$ sudo apt install python3-testresources

$ sudo apt-get install libtbb-dev

$ sudo apt-get install libatlas-base-dev gfortran

$ sudo apt-get install libprotobuf-dev protobuf-compiler

$ sudo apt-get install libgoogle-glog-dev libgflags-dev

#optional
$ sudo apt-get install libgtk-3-dev

#optional
$ sudo apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev
```

## 2 - Install cuda 11.8

```bash

$ wget https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/cuda-${OS}.pin


$ sudo mv cuda-${OS}.pin /etc/apt/preferences.d/cuda-repository-pin-600

$ wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda-repo-${OS}-11-8-local_11.8.0-520.61.05-1_amd64.deb

$ sudo dpkg -i cuda-repo-${OS}-11-8-local_11.8.0-520.61.05-1_amd64.deb

$ sudo cp /var/cuda-repo-${OS}-11-8-local/cuda-*-keyring.gpg /usr/share/keyrings/

$ sudo apt-get update

$ sudo apt-get -y install cuda

```

Then run nvidia-smi to see something like:

```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 520.61.05    Driver Version: 520.61.05    CUDA Version: 11.8     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:01:00.0 Off |                  N/A |
| N/A   54C    P3    20W /  N/A |      5MiB /  6144MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1465      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+

```

If not, you may need to reboot your computer.

## 3 - Install cudnn libraries

```bash

$ wget https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/cuda-${OS}.pin 

$ sudo mv cuda-${OS}.pin /etc/apt/preferences.d/cuda-repository-pin-600

$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/3bf863cc.pub

$ sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/ /"

$ sudo apt-get update

$ sudo apt-get install libcudnn8=${cudnn_version}-1+${cuda_version}

$ sudo apt-get install libcudnn8-dev=${cudnn_version}-1+${cuda_version}

```
## 4 - Install OpenCV 4.5.4

```bash

$ cd ~

$ wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.5.4.zip

$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.5.4.zip

$ unzip opencv.zip && unzip opencv_contrib.zip

$ cd opencv-4.5.4

$ mkdir build && cd build

$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_CUDA=ON \
-D BUILD_opencv_cudacodec=OFF \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=${cuda_arch} \
-D WITH_V4L=ON \
-D WITH_QT=OFF \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.5.4/modules \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D INSTALL_C_EXAMPLES=OFF \
-D BUILD_EXAMPLES=OFF ..

$ nproc

#regarding to your number of proc
$ make -j4

$ sudo make install

$ sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'

$ sudo ldconfig

```

## 5 - Makefile example including opencv

```makefile
CC = g++
OPENCV = `pkg-config opencv --cflags --libs`
EXE = app

all: main clean

main: main.o
        $(CC) -o ${EXE} main.o ${OPENCV}

main.o: main.cpp 
        $(CC) -c main.cpp ${OPENCV}

clean:
        rm -Rf *.o

```


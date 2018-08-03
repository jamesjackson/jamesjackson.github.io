---
layout: post
title: Ubuntu 16.04 Deep Learning Environment on Dell G7 Laptop (GTX1060 GPU)
comments: true
---

**Prepare Ubuntu on USB stick**

Download Ubuntu 16.04 iso:

http://releases.ubuntu.com/16.04/ubuntu-16.04.4-desktop-amd64.iso

Install Rufus on Windows:

https://rufus.akeo.ie/

Insert USB stick, open Rufus, select downloaded ISO, follow all defaults and recommendations.


**Set boot params and install Ubuntu**


Insert USB stick in G7, reboot, hit F12 during boot, enter BIOS, disable Secure Boot, save, exit. 

Hit F12 during boot, select boot from USB (UEFI), highlight "Try Ubuntu without installing" and press E to edit. Add nouveau.modeset=0 to the end of the linux line, press F10 to boot.

Ubuntu will start up but WiFi will not be working. Connect via ethernet port as a temporary workaround. Click the desktop icon to install Ubuntu. Be sure to select the correct drive (in this case the 128GB SSD), reboot when prompted.

Hit Esc during boot to bring up the GRUB menu, highlight Ubuntu, press E to edit, add nouveau.modeset=0 to the end of the linux line, press F10 to boot.


**Install the Nvidia drivers**

Press Ctrl-Alt-F4 to open a new console. Stop the X server, install drivers and reboot:

```
sudo service lightdm stop
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-get install nvidia-390
sudo reboot
```

Check the status:

```
nvidia-smi

Sat Jul 28 01:19:24 2018       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.77                 Driver Version: 390.77                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 106...  Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   44C    P3    19W /  N/A |    408MiB /  6078MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1092      G   /usr/lib/xorg/Xorg                           193MiB |
|    0      1926      G   compiz                                       143MiB |
|    0      2319      G   ...-token=20953B506B7716C886CFA3FDF016DE44    68MiB |
+-----------------------------------------------------------------------------+
```


**Install Cuda 9.0**

Download latest Cuda 9.0 (required by Tensorflow) runfile installer (local):

https://developer.nvidia.com/cuda-90-download-archive


Extract Cuda and install:

```
chmod +x cuda_9.0.176_384.81_linux.run
./cuda_9.0.176_384.81_linux.run --extract=$HOME
cd ~
sudo ./cuda-linux.9.0.176-22781540.run
```


Update env. by adding lines to .bashrc:

```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
source .bashrc
```


**Install CuDNN**


Download latest cuDNN for CUDA 9.0 (tarball):

https://developer.nvidia.com/rdp/cudnn-archive

Extract cuDNN and copy files:

```
gunzip cudnn-9.0-linux-x64-v7.1.tgz 
tar -xvf cudnn-9.0-linux-x64-v7.1.tar 

sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h
```

**Install Anaconda**

Download Anaconda:
https://www.anaconda.com/download/#linux

Install Anaconda:

```
chmod +x Anaconda3-5.2.0-Linux-x86_64.sh 
./Anaconda3-5.2.0-Linux-x86_64.sh
```

**Setup Conda env and Tensorflow**


TF URLs: https://www.tensorflow.org/install/install_linux#the_url_of_the_tensorflow_python_package

```
conda create -n tensorflow pip python=3.6
source activate tensorflow
pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.9.0-cp36-cp36m-linux_x86_64.whl
```

**Check that Tensorflow uses GPU**


```
>>> import tensorflow as tf
>>> sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
2018-07-27 23:59:40.673738: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
2018-07-27 23:59:40.829193: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:897] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2018-07-27 23:59:40.829919: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1392] Found device 0 with properties: 
name: GeForce GTX 1060 with Max-Q Design major: 6 minor: 1 memoryClockRate(GHz): 1.48
pciBusID: 0000:01:00.0
totalMemory: 5.94GiB freeMemory: 5.44GiB
2018-07-27 23:59:40.829951: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1471] Adding visible gpu devices: 0
2018-07-27 23:59:41.071679: I tensorflow/core/common_runtime/gpu/gpu_device.cc:952] Device interconnect StreamExecutor with strength 1 edge matrix:
2018-07-27 23:59:41.071725: I tensorflow/core/common_runtime/gpu/gpu_device.cc:958]      0 
2018-07-27 23:59:41.071733: I tensorflow/core/common_runtime/gpu/gpu_device.cc:971] 0:   N 
2018-07-27 23:59:41.071961: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1084] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 5212 MB memory) -> physical GPU (device: 0, name: GeForce GTX 1060 with Max-Q Design, pci bus id: 0000:01:00.0, compute capability: 6.1)
Device mapping:
/job:localhost/replica:0/task:0/device:GPU:0 -> device: 0, name: GeForce GTX 1060 with Max-Q Design, pci bus id: 0000:01:00.0, compute capability: 6.1
2018-07-27 23:59:41.134857: I tensorflow/core/common_runtime/direct_session.cc:288] Device mapping:
/job:localhost/replica:0/task:0/device:GPU:0 -> device: 0, name: GeForce GTX 1060 with Max-Q Design, pci bus id: 0000:01:00.0, compute capability: 6.1
```




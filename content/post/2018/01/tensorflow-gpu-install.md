+++
categories = ["技术"]
date = "2018-01-15T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "tensorflow-gpu install"

+++

# centos7下安装tensorflow-gup 

cuda8、cudnn8、tensorflow-gup安装过程：

### 1、下载相关软件

下载cuda8.0

wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run

下载cudnn8.0-v6.0

wget http://developer2.download.nvidia.com/compute/machine-learning/cudnn/secure/v6/prod/8.0_20170427/cudnn-8.0-linux-x64-v6.0.tgz?yQ3tYwXxUZLlPzHDcsdoKYsza8ARGpYxi6k1Knhuhr-ZOKb5z4schgCD8kJF8LKCbYKRFniqc3_5rMpPPsZlGz1EBe_5GKe3KevXpgG6-Q0akSJByfXqqB0lH1wsl1c1rBJ4eZzVgX9GuGZpt5oXDMEjnwJoPGFLE7OaWBgMSpABsTdq8iuhumtn9ZV6pRByDN7JKbZD


### 2、安装cuda8.0
sh cuda_8.0.61_375.26_linux-run
```
Do you accept the previously read EULA?
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 387.26?
(y)es/(n)o/(q)uit: y

Install the CUDA 8.0 Toolkit?
(y)es/(n)o/(q)uit: y

Enter Toolkit Location
 [ default is /usr/local/cuda-8.0 ]:

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: y

Install the CUDA 8.0 Samples?
(y)es/(n)o/(q)uit: y

Enter CUDA Samples Location
 [ default is /root ]:

Installing the CUDA Toolkit in /usr/local/cuda-8.0 ...

Missing recommended library: libGLU.so
Missing recommended library: libX11.so
Missing recommended library: libXi.so
Missing recommended library: libXmu.so
Missing recommended library: libGL.so


 Installing the CUDA Samples in /root ...
Copying samples to /root/NVIDIA_CUDA-8.0_Samples now...
Finished copying samples.

= Summary = 

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-8.0
Samples:  Installed in /root, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-8.0/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-8.0/lib64, or, add /usr/local/cuda-8.0/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run the uninstall script in /usr/local/cuda-8.0/bin

Please see CUDA_Installation_Guide_Linux.pdf in /usr/local/cuda-8.0/doc/pdf for detailed information on setting up CUDA.

Logfile is /tmp/cuda_install_10321.log
```



### 3、设置环境变量
```
vim /etc/profile
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
export CUDA_HOME=/usr/local/cuda
```
### 4、测试cuda
```
cd /usr/local/cuda/samples/1_Utilities/deviceQuery
make
./deviceQuery 
```
```
deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 4, Device0 = Tesla P40, Device1 = Tesla P40, Device2 = Tesla P40, Device3 = Tesla P40
Result = PASS
```

### 5、安装cudnn8.0-v6.0
```
tar -zxf cudnn-8.0-linux-x64-v6.0.tgz
cd  cuda
cp -P lib64/* /usr/local/cuda/lib64/
cp -P include/* /usr/local/cuda/include/
chmod a+r /usr/local/cuda/lib64/libcudnn*
注：-P参数是为了保留软链接
```


### 6、安装tensorflow-gpu
```
yum install python-pip python-dev
pip install --upgrade pip
pip install --upgrade tensorflow-gpu 
```
### 7、验证tensorflow
```
>>> import tensorflow
>>> import tensorflow as tf
>>> tf.__version__
>>> '1.4.1'
>>> tf.__path__
>>> ['/usr/lib/python2.7/site-packages/tensorflow']
```



### 8、相关问题：

卸载cuda：
```
/usr/local/cuda-8.0/bin/uninstall_cuda_8.0.pl
```
卸载nvidia driver:
```
/usr/bin/nvidia-uninstall
```
### 9、相关检查命令
```
nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Nov__3_21:07:56_CDT_2017
Cuda compilation tools, release 9.1, V9.1.85
```

```
nvidia-smi
Mon Jan 15 10:05:48 2018       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 375.26                 Driver Version: 375.26                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla P40           Off  | 0000:02:00.0     Off |                    0 |
| N/A   21C    P0    49W / 250W |      0MiB / 22912MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla P40           Off  | 0000:03:00.0     Off |                    0 |
| N/A   22C    P0    49W / 250W |      0MiB / 22912MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  Tesla P40           Off  | 0000:83:00.0     Off |                    0 |
| N/A   23C    P0    48W / 250W |      0MiB / 22912MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  Tesla P40           Off  | 0000:84:00.0     Off |                    0 |
| N/A   23C    P0    46W / 250W |      0MiB / 22912MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+
```


### 10、安装cuda参考

http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile

+++
categories = ["技术"]
date = "2018-09-22T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "TFServing-gpu deploy"
+++

# TFServing gpu 服务环境搭建 

TFServing_docker-ce_nvidia-docker_nvidia-dirver_cuda_cudnn安装过程：

### 1、docker-ce install

from:https://stackoverflow.com/questions/42981114/install-docker-ce-17-03-on-rhel7/45033117#45033117

```
#yum remove docker docker-common container-selinux docker-selinux docker-engine
yum install -y yum-utils
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
subscription-manager repos --enable=rhel-7-server-extras-rpms
yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.55-1.el7.noarch.rpm
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl restart docker
systemctl status docker.service
```

### 2、nvidia-docker install 

from:https://blog.csdn.net/itaacy/article/details/72628792

```
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker-1.0.1-1.x86_64.rpm
rpm -i /tmp/nvidia-docker*.rpm && rm /tmp/nvidia-docker*.rpm
systemctl start nvidia-docker
#check
nvidia-docker run --rm nvidia/cuda nvidia-smi
```

### 3、cuda install

```
#https://developer.nvidia.com/cuda-toolkit-archive
wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda_9.0.176_384.81_linux-run

sh cuda_9.0.176_384.81_linux-run

#安装过程中: 
#Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 384.81？ (y)es/(n)o/(q)uit:n
#注意：此步选择y，其余选y或者default即可
```


### 4、cudnn install
```
#先从官方下载cudnn，解压后进行如下操作
cp cudnn/include/cudnn.h /usr/local/cuda-9.0/include
cp cudnn/lib64/libcudnn* /usr/local/cuda-9.0/lib64
chmod a+r /usr/local/cuda-9.0/include/cudnn.h /usr/local/cuda-9.0/lib64/libcudnn*
```


### 5、TFServing deploy

启动TFServing:

```
#模型路径 /root/predictproductword 
nvidia-docker run -p 10001:8501 -p 10000:8500 -v /root/predictproductword:/models/predictproductword -e MODEL_NAME=predictproductword -t tensorflow/serving:latest-gpu
```

启动日志：

```
 08:53:40.758380: 
 I tensorflow_serving/core/loader_harness.cc:86] Successfully loaded servable version {name: predictproductword version: 1}

 08:53:40.806719: 
 I tensorflow_serving/model_servers/main.cc:327] Running ModelServer at 0.0.0.0:8500 ...

[warn] getaddrinfo: address family for nodename not supported

 08:53:40.818440: 
 I tensorflow_serving/model_servers/main.cc:337] Exporting HTTP/REST API at:localhost:8501 ...

[evhttp_server.cc : 235] RAW: Entering the event loop ...
```

服务测试：
```
 curl -H "Content-Type:application/json" -X POST --data '{"instances":[{"input_sentence_lengths":57,"input_sentences":[355,51,17,223,478,120,361,2035,2035,2035,2035,8,2035,2035,2035,8,9,35,36,12,37,38,39,40,41,8,2035,42,2035,2035,2035,2035,8,607,180,1035,8,9,46,47,8,48,49,50,47,51,17,2035,2035,8,12,52,12,53,40,41,53,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]}],"signature_name":"predict_tags"}'  http://localhost:10001/v1/models/predictproductword:predict

返回结果：
 
{
    "predictions": [[0, 0, 0, 0, 0, 1, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ]
}

```



###  6、相关问题及解决：

####  1、Error: unsupported CUDA version: driver 8.0 < image 9.0.176

```
$nvidia-docker run --rm nvidia/cuda nvidia-smi
Status: Downloaded newer image for nvidia/cuda:latest

nvidia-docker | 2018/09/15 16:07:00 Error: unsupported CUDA version: driver 8.0 < image 9.0.176

解决：https://github.com/NVIDIA/nvidia-docker/issues/497

升级nivida显卡的驱动driver nvidia-384

You need at least a 384.xx series driver for CUDA 9. NVIDIA recommends

384.81 or later.

If you're installing from deb packages, note that the package name is

different for nvidia-384 versus nvidia-375 in order that the major driver

version upgrade is intentional rather than automatic.

$nvidia-smi

 
其实只需要在安装cuda的时候同步安装驱动即可,解决此问题

sh cuda_9.0.176_384.81_linux-run
安装过程中: 
Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 384.81？ (y)es/(n)o/(q)uit:n

注意：此步选择y，其余选y或者default即可

```



#### 2、NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running. 

```
解决此问题步骤：

1.1 查看显卡状态
$ lspci | grep -i nvidia  可见4块显卡，正常
1.2 检测安装包无误
$ md5sum cuda_9.0.176_384.81_linux-run
24fea3b7f2e5f7e3f155cd73bc008108   
与官网的checksum（http://developer.download.nvidia.com/compute/cuda/9.0/Prod/docs/sidebar/md5sum.txt）对比，无误。

1.3 检查系统依赖
$ yum info dkms
$ yum info libvdpau 
$ yum info kernel-devel

1.4 为内核安装nvdia模块

dkms的模块需要经过added, build, install 3个步骤才能被modinfo检测到
$ dkms status
nvidia, 384.81: added
显然，nvidia模块在安装的时候只是被added，还没有生成installed模块，原因不详。

# dkms build -m nvdia -v 384.81
会报kernel headers not found的错误，大概就是找不到/lib/3.10.0-693.11.6.el7.x86_64/build
如果我们
$ cd /var/lib/dkms/nvidia/384.81/source/ 
$ make
会报/lib/modules/3.10.0-693.11.6.el7.x86_64/build: 没有那个文件或目录的错误

解决：
查看现有依赖：
$ ls /var/lib/dkms/nvidia
384.81  kernel-3.10.0-693.11.6.el7.x86_64-x86_64
$ ls /usr/src/kernels/
3.10.0-327.28.3.el7.x86_64  3.10.0-693.11.6.el7.x86_64
如果没有3.10.0-693.11.6.el7.x86_64，可以进行下载及安装
wget ftp://ftp.pbone.net/mirror/ftp.scientificlinux.org/linux/scientific/7.0/x86_64/updates/security/kernel-devel-3.10.0-693.11.6.el7.x86_64.rpm
rpm -i kernel-devel-3.10.0-693.11.6.el7.x86_64.rpm

# dkms build -m nvidia -v 384.81
# dkms install -m nvidia -v 384.81

可能需要重启
$ reboot
$ modinfo nvidia
$ modinfo nvidia-uvm

$dkms status
nvidia, 384.81, 3.10.0-693.11.6.el7.x86_64, x86_64: installed

问题解决~

from:
https://zhuanlan.zhihu.com/p/32958364
https://www.jianshu.com/p/127e019ed2e8
https://blog.csdn.net/yijuan_hw/article/details/53439408
```
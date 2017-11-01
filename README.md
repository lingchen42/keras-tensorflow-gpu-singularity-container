# keras-tf-gpu-singularity-container
Here are the steps to build a singularity container for keras with tf-gpu on a local computer for use on a cluster. 

## Set up a ubuntu virtual machine with vagrant and virtualbox
1. Install [virtualbox](https://www.virtualbox.org/wiki/Download_Old_Builds), use v5.1 or older versions, vagrant 2.0.0 is not compatible with v5.2.
2. Install [vagrant 2.0.0](https://www.vagrantup.com/downloads.html).
3. Install [vagrant manager](http://vagrantmanager.com/downloads/).
4. Install vagrant unbuntu vm 14.04. 
   ```
   mkdir ~/ubuntu-vm
   cd ~/ubuntu-vm
   vagrant init ubuntu/trusty64 --box-version 14.04
   ```

## Set up singularity inside ubuntu virtual machine,
1. Enter the ubuntu virtual machine.
    ```
    cd ~/ubuntu-vm
    vagrant up
    vagrant ssh
    ```
2. Inside ubuntu vm, install singularity 2.3.1
    ```
    VERSION=2.3.1
    wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
    tar xvf singularity-$VERSION.tar.gz
    cd singularity-$VERSION
    ./configure --prefix=/usr/local
    make
    sudo make install
    ```

## Build singularity container (all done inside ubuntu-vm)
1. Check NVIDIA driver version on the cluster, download the same version driver from NVIDIA website.
    ```
    nvidia-smi
    ```
    For me, I got:
    ```
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 384.59                 Driver Version: 384.59                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  TITAN X (Pascal)    On   | 00000000:02:00.0 Off |                  N/A |
    | 39%   67C    P2   113W / 250W |  11636MiB / 12189MiB |     35%      Default |
    +-------------------------------+----------------------+----------------------+
    |   1  TITAN X (Pascal)    On   | 00000000:03:00.0 Off |                  N/A |
    | 23%   33C    P8    16W / 250W |      2MiB / 12189MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    |   2  TITAN X (Pascal)    On   | 00000000:82:00.0 Off |                  N/A |
    | 33%   57C    P2    63W / 250W |  11636MiB / 12189MiB |     20%      Default |
    +-------------------------------+----------------------+----------------------+
    |   3  TITAN X (Pascal)    On   | 00000000:83:00.0 Off |                  N/A |
    | 31%   54C    P2    60W / 250W |  11636MiB / 12189MiB |     21%      Default |
    +-------------------------------+----------------------+----------------------+
    ```
    So I downloaded [NVIDIA-Linux-x86_64-384.59](http://us.download.nvidia.com/XFree86/Linux-x86_64/384.59/NVIDIA-Linux-x86_64-384.59.run). It is a bit tricky to find it.\
    But it should be http://us.download.nvidia.com/XFree86/Linux-x86_64/$VERSION/NVIDIA-Linux-x86_64-$VERSION.run

2. Check cuda version on the cluster, download the corresponding cuda driver from NVIDIA website.
    ```
    nvcc --version
    ```
    For me, I got:
    ```
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2016 NVIDIA Corporation
    Built on Tue_Jan_10_13:22:03_CST_2017
    Cuda compilation tools, release 8.0, V8.0.61
    ```
    So I downloaded [cuda_8.0.61](https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run) from https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run.
    I found it to be OK that if I'm using a run file but the nvidia version of cuda is different from that on the cluster (375.26 != 384.59).

3. Store the downloaded files and above scripts under the same folder
4. Run `sh build.sh`.
5. Copy the resulting `tensorflow_gpu-1.1.0-cp27-linux_x86_64.img` onto the cluster. 
    [vagrant scp](https://github.com/invernizzi/vagrant-scp) can be used to copy files in vm outside. 
6. Running the container on cluster
```
singularity shell --nv tensorflow_gpu-1.1.0-cp27-linux_x86_64.img
```
    
## Other notes
1. This is adapted from https://github.com/jdongca2003/Tensorflow-singularity-container-with-GPU-support.
2. This assumes that the nvidia driver is installed on the cluster.
3. When building the ubuntu-vm, give it at least 2GB memory. Otherwise, the installation of some python libraries may fail.

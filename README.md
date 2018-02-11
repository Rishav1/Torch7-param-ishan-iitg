# Torch7 installation on Param Ishan HPC at IITG
How to set up Torch7 with CudNN in IIT Guwahati's Param Ishan High Performing Cluster.

Param Ishan has support for only CUDA 7.5 as of now, and therefore, only CuDNN v5 is possible.

## Requirements
1. Cuda 7.5 with cuDNN
2. Anaconda with Python3
3. Great amount of Patience

## Note
I am creating this from my memory, so if you find that this fails somewhere, ping me. I will subsequently add the change here.

## Steps on gpu-login node.

1. Copy Cuda7.5 and install CuDNN locally in your home directory.
```bash
	mkdir cuda75
	cp -r /cm/shared/apps/cuda75/toolkit/7.5.18/* cuda75/
	#Download CuDNN v5.1 from [Nvidia website](https://developer.nvidia.com/rdp/cudnn-download)
	tar -xsvf cudnn-7.5-linux-x64-v5.1.tgz
	mv cuda/include/cudnn.h cuda75/include/
	mv cuda/lib65/* cuda75/lib64/
	rm -rf cuda
```

2. Install Anaconda(Python 2.7 version) locally.
```bash
# Download install script from [Anaconda website](https://www.anaconda.com/download/#linux)
chmod +x Anaconda2-5.0.1-Linux-x86_64.sh
./Anaconda2-5.0.1-Linux-x86_64.sh 
```


3. Set appropriate modules in bashrc.
```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

# User specific aliases and functions
module load gcc/5.2.0
module load slurm
module load module-git
#Unload all Intel modules
module load intel/32-all
module load intel/64-all
module unload intel/32-all
module unload intel/64-all

# Change proxy setting.
export http_proxy=http://USER:PASS@202.141.80.22:3128/
export https_proxy=$http_proxy
export LC_COLLATE=C
export LC_ALL=C

# CUDA settings
export CUDA_ROOT=~/cuda75
export CUDA_HOME=~/cuda75
export PATH=${CUDA_HOME}/bin:$PATH
export LD_LIBRARY_PATH=${CUDA_HOME}/lib64:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/cm/shared/apps/glibc_2.14:$LD_LIBRARY_PATH

# NVIDIA driver
export LD_LIBRARY_PATH=/cm/local/apps/cuda-driver/libs/352.79/lib64:$LD_LIBRARY_PATH

# Anaconda Variables and includes
export PATH=/home/$USER/anaconda3/bin:$PATH
export LD_LIBRARY_PATH=/home/$USER/anaconda3/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=/home/$USER/anaconda3/lib:$LIBRARY_PATH
export INCLUDEPATH=/home/$USER/anaconda3/include:$INCLUDEPATH
```
After editing the bashrc, remember to source it using
```bash
source ~/.bashrc
```


4. Anaconda Installation of required packages and executable for Torch7(You may require some more packages that I don't remember now. Install them like this when missing error pops up).
```bash
conda create --name torch7-env python=3.5
activate torch7-env
conda install cmake
conda install matplotlib -c conda-forge
conda install readline
```

5. Clone Torch7 distro installation from [github](https://github.com/torch/torch7)
```bash
#Your initial git config
git config --global user.name "NAME"
git config --global user.email "EMAIL"
git config --global http.proxy "http://USER:PASS@202.141.80.22:3128/"
git config --global https.proxy "https://USER:PASS@202.141.80.22:3128/"
git config --global url."https://".insteadOf git://
```

Clone Torch7
```bash
git clone https://github.com/torch/distro.git
cd distro
```

Modify the install.sh script to use the anaconda libraries.
```
# Edit the following lines of install.sh in the torch7 distro.
nano install.sh
# Comment out lines 39 to 41.

OLDPATH=$PATH

#if [[ $(echo $PATH | grep conda) ]]; then
#    export PATH=$(echo $PATH | tr ':' '\n' | grep -v "conda[2-9]\?" | uniq | tr '\n' ':')
#fi

echo "Prefix set to $PREFIX"

```

6. Now run the install script with LUA5.1. LUAJIT 2.0/2.1 as well as LUA 5.2 build are not supported by the architecture.

```bash
export CC=/cm/shared/apps/gcc/5.2.0/bin/gcc
export CXX=/cm/shared/apps/gcc/5.2.0/bin/g++
TORCH_LUA_VERSION=LUA51 ./install.sh
```

7. After successful installation, test it by running the following command.
```bash
th> Do you really want to exit ([y]/n)? y
[r.chourasia@gpu-login ~]$ th
 
  ______             __   |  Torch7 
 /_  __/__  ________/ /   |  Scientific computing for Lua. 
  / / / _ \/ __/ __/ _ \  |  Type ? for help 
 /_/  \___/_/  \__/_//_/  |  https://github.com/torch 
                          |  http://torch.ch 
	
th> cudnn = require 'cudnn'
                                                                      [4.1991s]	
th> 
```

Cheers! You are all set up.




























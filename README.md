# MiniMUC: A log of my mini hpc system at home

This is just to keep track of the changes I introduce to my mini hpc system at home. I decided to get a workstation and use it as a mini hpc to learn about these systems in linux.

## Installation:

I installed OpenSuse Tumbleweed to have access to the most recent kernel and drivers and be able to use the GPU (Intel ARC 770) for computational purposes.

## Post-install changes:

The following packages have been added:

```sh

zypper in intel-gpu-tools
zypper in git
zypper in -t pattern devel_basis

```

## Avoid GDM to send the computer to sleep when no user logs in

Avoid `gdm` to suspend the computer while logged in via `ssh`. I keep getting a `suspend` message after some time... I log in via `ssh`, which means `gdm` is waiting for a user to log in and send the system to sleep... To fix this (which is really annoying):

```sh
su - gdm -s /bin/bash
#see current settings:
dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type
dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout

#set new time out to 0 (zero), meaning never
dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
exit
```

## Installing mini-forge

Download mini-forge and install it...


# Install Rstudio server

```sh
zypper install apache2
systemctl start apache2
systemctl enable apache2

firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload

zypper install rstudio-server
systemctl start rstudio-server
systemctl enable rstudio-server

firewall-cmd --zone=public --add-port=8787/tcp --permanent
firewall-cmd --reload

zypper install gcc-fortran libcurl-devel xz-devel libbz2-devel libjpeg8-devel libpng16-devel
```



## GPU...
I have had lots of problems with this...

I included the Intel repo and installed the following packages:

As root:

```sh
zypper install -t pattern devel_C_C++
zypper install kernel-devel

zypper addrepo https://yum.repos.intel.com/oneapi oneAPI
rpm --import https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB

zypper install intel-basekit-2024.0

```

But keep getting the following warning:

```
Warning:  the following driver(s) were not found loaded in the kernel:  sep5.
Warning:  no vtsspp driver was found loaded in the kernel.
Warning:  no socwatch driver was found loaded in the kernel.
Warning:  the following driver(s) were not found loaded in the kernel:  socperf3.
The PAX service is not loaded anymore.
sep5.service failing...

```

And nothing seems to work with the GPU... The error seems to be caused by the compilation of socwatch failing in kernel 6.

So, I ended up unintalling all the intel packages to try to compile the NEO package myself. For this:

```sh
mkdir intel
cd intel
mkdir src
cd src

git clone https://github.com/intel/vc-intrinsics.git vc-intrinsics
git clone -b llvmorg-14.0.5 https://github.com/llvm/llvm-project.git llvm-project
git clone -b ocl-open-140 https://github.com/intel/opencl-clang.git llvm-project/llvm/projects/opencl-clang
git clone -b llvm_release_140 https://github.com/KhronosGroup/SPIRV-LLVM-Translator.git llvm-project/llvm/projects/llvm-spirv
git clone https://github.com/KhronosGroup/SPIRV-Tools.git SPIRV-Tools
git clone https://github.com/KhronosGroup/SPIRV-Headers.git SPIRV-Headers
git clone https://github.com/intel/intel-graphics-compiler.git igc
git clone https://github.com/intel/compute-runtime.git neo
git clone https://github.com/intel/metrics-library.git

#I had to install the following packages because I kept getting compilation time errors:
su
zypper install libva-devel gmmlib-devel libigc-devel libigdfcl-devel libigfxcmrt-devel python311-Mako

mkdir build_neo
cd build_neo
cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/sergio.vargas/Repos/intel -DCMAKE_BUILD_TYPE=Release ../neo/
make -j 28
#I needed to run make install as root to allow for the creation of a file in /etc/OpenCL
su
make install

mkdir build_igc
cd build_igc
cmake -DCMAKE_INSTALL_PREFIX:PATH=/home/sergio.vargas/Repos/intel -DCMAKE_BUILD_TYPE=Release ../igc/
make -j 28
make install

```



To compile `beagle-lib`:

```sh

zypper in cmake
zypper in java-21-openjdk-devel
zypper in ocl-icd-devel

git clone --depth=1 https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=$HOME -DBUILD_CUDA=OFF ..
make
make install

```




# Install ollama + llama3

zypper install intel-oneapi-pytorch intel-oneapi-mkl intel-oneapi-runtime-mkl
zypper install ollama

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/oneapi/mkl/2024.0/lib:/opt/intel/oneapi/compiler/2024.1/lib




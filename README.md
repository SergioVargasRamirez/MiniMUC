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

With this configuration beagle compiles but the I get a run time error.

installed: vulkan-tools

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


## Installing intel one-api
As root:

```sh
zypper install -t pattern devel_C_C++
zypper install kernel-devel

zypper addrepo https://yum.repos.intel.com/oneapi oneAPI
rpm --import https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB

zypper install intel-basekit
intel-basekit-2024.0

```

Getting the following warning:

```
Warning:  the following driver(s) were not found loaded in the kernel:  sep5.
Warning:  no vtsspp driver was found loaded in the kernel.
Warning:  no socwatch driver was found loaded in the kernel.
Warning:  the following driver(s) were not found loaded in the kernel:  socperf3.
The PAX service is not loaded anymore.
sep5.service failing...

```

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


# Install ollama + llama3

zypper install intel-oneapi-pytorch intel-oneapi-mkl intel-oneapi-runtime-mkl
zypper install ollama





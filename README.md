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
zypper in intel-opencl intel-opencl-devel opencl-cpp-headers ocl-icd-devel clinfo

git clone --depth=1 https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=$HOME -DBUILD_CUDA=OFF ..
make
make install

```

With this configuration beagle compiles but the I get a run time error.

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









# MiniMUC: A log of my mini hpc system at home

This is just to keep track of the changes I introduce to my mini hpc system at home. I decided to get a workstation and use it as a mini hpc to learn about these systems in linux.

## Installation:

I installed OpenSuse Tumbleweed to have access to the most recent kernel and drivers and be able to use the GPU (Intel ARC 770) for computational purposes. After many attempts to get the GPU working with different programs I was pointed to an incompatibility between the kernel version >6.8 and the intel-oneapi packages. I then installed Ubuntu 22LTS to test whether the intel packages worked with the Arc 770 GPU. This test was successful. Therefore, I installed OpenSuse Leap 15.6 RC, which comes with kernel 6.4 with the necessary backport updates to support Arc 770. I installed OpenSuse to follow the suggestion to avoid modifying the host system and get all the necessary GPU packagery running via containers. This actually works very well. I can use an Ubuntu 22LTS container and install the necessary intel-oneapi, etc. packages and use the container to run the GPU computation whenever I need them to do it.  

## Post-install changes:

The following packages have been added:

```sh

zypper in intel-gpu-tools
zypper in git-lfs
zypper in podman podman-docker docker-compose
zypper in htop

#software I previously installed but removed
#zypper in -t pattern devel_basis
#zypper in docker

```

## Change server name

Add name `miniMUC` to `/etc/hosts` and `/etc/hostname`.

## Mount RAID drive

Add `UUID=527495ce-1add-4a6b-98c2-3859c84aba23  /mnt/scratch            ext4   defaults                      0  0` to `fstab`

Create group `data` to assing `scratch` to it and allow access to the `scratch` folder.

## Testing GPU with docker's IPEX-LLM image

First, download some LLMs. I downloaded:

 - Meta-Llama3-8B-Instruct
 - Meta-Llama3-70B-Instruct
 - Mistral-7B-Instruct
 - chatglm2-6b

The LLMs are stored in the folder `~/LLModels`. To test the GPU via docker:

```sh
docker pull intelanalytics/ipex-llm-xpu
docker run -itd --net=host --privileged --device /dev/dri -v ~/LLModels:/llm/llm-models  --memory="32G" --name=ipex --shm-size="16g" intelanalytics/ipex-llm-xpu
docker exec -it ipex bash
```

This will start a session inside the container. In the container do the following to test the GPU:

```sh
cd llm
python chat.py --model-path llm-models/chatglm2-6b

#opens a chat-like terminal to interact with the llm

```

It is a good idea to have another terminal with `intel_gpu_top` running to check the GPU. I tried llama3-8B as well and llama3-70B but this last model needs to much RAM

## Testing GPU with Podman

An alternative to `docker` available in OpenSuse is `podman`. I would like to use this container manager instead of `docker`. Among other reasons, it runs fully in user space and doesn't use a deamon. Now, I have no idea whether it can run the GPU. The problem is that one cannot install `docker` and `podman` on the same host. In OpenSuse, `zypper` will ask whether to uninstall `docker` before installing `podman` or cancel. In principle, all should work replacing `docker` for `podman`.

```sh

podman pull intelanalytics/ipex-llm-xpu
#podman run -itd --net=host --privileged --device /dev/dri --volume ~/LLModels:/llm/llm-models  --memory="32G" --name=ipex --shm-size="16g" intelanalytics/ipex-llm-xpu
#with podman, I don't seem to need the --privileged flag
#I cannot run the container with the --memory or the --shm-size flags

podman run -itd --net=host --device /dev/dri --volume ~/LLModels:/llm/llm-models --name=ipex intelanalytics/ipex-llm-xpu
podman exec -it ipex bash

```

The GPU is also used by the LLMs with `podman` so I will keep using this method.

## Creating a GPU container with Podman

The tests above were done with available images. I would like to build my own images and, hopefully, make them minimal. As a test, I configured an image to compile [llama.cpp](https://github.com/ggerganov/llama.cpp) in it and run it in the container with my Arc 770 GPU.


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

#open port 8787 for rstudio-server
firewall-cmd --zone=public --add-port=8787/tcp --permanent
firewall-cmd --reload

#open port 8080 for marimo
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload

#Add `ServerName minimuc` to httpd.conf (Apache)

zypper install gcc-fortran libcurl-devel xz-devel libbz2-devel libjpeg8-devel libpng16-devel

#required for svglite
zypper install fontconfig-devel

#required by devtools
zypper install libopenssl-devel harfbuzz-devel fribidi-devel freetype-devel libpng-devel libtiff-devel libjpeg-devel

```

# Install marimo inside a podman pod

To test marimo, I use a container runing a miniconda3 image with port 8080 exposed and a directory "/notebooks" attached to it to ensure the notebooks are saved.

To be able to reach marimo from a remote, yet in the same network, PC via web browser Apache2 needs to be configured to:
  1. activate the **proxy** module
  2. forward port 8080 to the marimo podman pod

```sh
# to enable virtualhosts the module proxy needs to be active
# as root run:
a2enmod proxy
#
#
# VirtualHost template
# Note: to use the template, rename it to /etc/apache2/vhost.d/yourvhost.conf.
# Files must have the .conf suffix to be loaded.
#
# See https://en.opensuse.org/SDB:Apache_installation for further hints
# about virtual hosts.
#
# Almost any Apache directive may go into a VirtualHost container.
# The first VirtualHost section is used for requests without a known
# server name.
#
<VirtualHost *:8080>

    ServerName minimuc

    ProxyRequests Off
    ProxyPreserveHost On

    ProxyPass / http://0.0.0.0:8080/
    ProxyPassReverse / http://0.0.0.0:8080/

</VirtualHost>
```

To run the pod I have tested:
```sh
podman run -it --rm --net host -p 8080:8080 --volume ~/Repos/marimo/notebooks:/notebooks --replace --name=marimo b2537544abe6
```
I still need to check how to run the podman when the computer restarts and how to avoid having to connect with a session token. I have checked that the notebooks are successfully saved to the HD.

# Podman stuff

```sh
#remove "external" containers

array=($(podman ps -a --external | head -n -1 | awk 'BEGIN{FS=" "} {print $1}'))

for i in ${array[@]};do
 podman container rm $i
done

#list and remove images
podman image list
podman image rm d0c44b7baab3

#list and remove containers
podman container list
podman ps -a 

#build image from dockerfile
podman build --format docker -f ubuntu_22_gpu_beagle.beast2.dockerfile

#run image in interactive mode: will start a tty 
podman run -it --net=host --device=/dev/dri --name=beagle.beast2 0bf7f47588cc 

```

---

<p xmlns:cc="http://creativecommons.org/ns#" >This work by <span property="cc:attributionName">Sergio Vargas R.</span> is licensed under <a href="https://creativecommons.org/licenses/by/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">Creative Commons Attribution 4.0 International<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" alt=""><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" alt=""></a></p>

---

## Avoid GDM to send the computer to sleep when no user logs in

This was necessary with Tumbleweed but is somehow not needed with Leap.

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






## Beagle

To compile `beagle-lib` inside a podman container with the necessary intel packages for Arc 770 GPU interaction. Inside the container:

```sh
#the container includes the necessary intel packages, java and other beagle-lib requirements, beagle-lib, and beast2

source /opt/intel/oneapi/setvars.sh
cd beagle-lib
mkdir build
cd build
cmake ..
make
make install

#test the install; a GPU should be listed
cd /beast2/bin
./beast2 -beagle_info


```



```
#
# VirtualHost template
# Note: to use the template, rename it to /etc/apache2/vhost.d/yourvhost.conf.
# Files must have the .conf suffix to be loaded.
#
# See https://en.opensuse.org/SDB:Apache_installation for further hints
# about virtual hosts.
#
# Almost any Apache directive may go into a VirtualHost container.
# The first VirtualHost section is used for requests without a known
# server name.
#
<VirtualHost *:8050>

    ServerName minimuc

    ProxyRequests Off
    ProxyPreserveHost On

    ProxyPass / http://0.0.0.0:8050/
    ProxyPassReverse / http://0.0.0.0:8050/

</VirtualHost>
```



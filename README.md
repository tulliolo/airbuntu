# AirBuntu
How to build a custom kernel in order to keep your AirGap PC offline.

## Introduction
This tutorial shows how to build a custom kernel in order to remove any network interface.
Having an offline PC can be useful for example in order to implement a cold wallet (e.g. [Electrum](https://electrum.org)). Such goal could be achieved in different ways, e.g. by disabling the network manager, or blacklisting kernel modules; with this tutorial we will perform something stronger: instead of _disabling_ the ability to go online, we will completely _erase_ it.

This guide has been written and tested for Ubuntu, but il could be extended, with proper (and out of scope) modifications, to any linux distribution.

:warning: **WARNING: the following actions are potentially destructive, if not properly done. We strongly suggest to use a fresh installation, or to backup all the important data before to proceed.**


## System setup
The OS installation is out of scope in this work. We used a minimal [Ubuntu 22.04 LTS](https://ubuntu.com/#download) installation without third party drivers.

Open a terminal and perform a system upgrade with:

```
sudo apt update && sudo apt upgrade -y
```

If you want to use AppImages (e.g. for Electrum) install libfuse2 with:

```
sudo apt install libfuse2
```

Now you can download, verify the signatures and adjust permission for the [Electrum AppImage](https://download.electrum.org/4.3.4/electrum-4.3.4-x86_64.AppImage) (the detailed procedure is out of scope).

Install the needed dependencies to build the kernel with:

```
sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
```

Now, check the network status with:

```
ip link
ip addr
```

![screenshot](/screenshots/network_status_start.png?raw=true)

In the screenshot above I have only one active interface (_enp0s3_), with an assigned IP address; but you could have any other wired and/or wireless interface. The goal is to make them all disappear.

## Kernel installation
### Kernel download
In the terminal, create a new directory for saving the kernel source code with the commands:

```
cd
mkdir src
cd src
```

Check the current running kernel version with:

```
uname -mrs
```

In my case (see screenshot below) I'm running a kernel of serie Linux 5.19.x.<br/>
Now, look for an [available kernel](https://cdn.kernel.org/pub/linux/kernel/v5.x) of the same serie (in my case 5.19.17) and download it with:

```
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.17.tar.xz
```

:warning: **WARNING**: replace _linux-5.19.17.tar.xz_ with any version of your choice, but be sure to grab a kernel of the same serie of your running one! Indeed, in the following steps we will copy the current running configuration in order not to configure the new kernel from scratch. Kernels of different series could have differences in the configuration files, which could be not totally compatible.

:warning: **WARNING**: in the following steps I would refer only to the kernel of my choice (5.19.17). If you grab something else, be careful to properly adapt the following commands.

![screenshot](/screenshots/kernel_download.png?raw=true)

### Kernel configuration
Now, we can unzip the new kernel file, enter in it's directory, and then copy in it the current running kernel configuration. We will also make a backup copy of it... just in case...:

```
tar xvf linux-5.19.17.tar.xz
cd linux-5.19.17
cp /boot/config-$(uname -r) .config
cp .config .config.back
```

![screenshot](/screenshots/config_copy.png?raw=true)

Enter in the kernel configuration tool with the following command. It will automatically load the current running kernel configuration, saved in the previous step:

```
make menuconfig
```

The command will show the kernel configuration tool, as in the following picture:

![screenshot](/screenshots/config_edit.png?raw=true)

The kernel configuration options can be navigated using the :arrow_up: arrow up and :arrow_down: arrow down keys_ , while the underneath commands (_Select_, _Exit_, _Save_, etc...) can be navigated with the :arrow_left: arrow left and :arrow_right: arrow right keys.

The fields with a right arrow (--->) are submenu, which can be accessed by typing the :leftwards_arrow_with_hook: enter key (with the _Select_ option highlighted).

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

## Kernel download
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

The kernel configuration options can be selected using the :arrow_up: arrow up and :arrow_down: arrow down keys, while the underneath actions (**Select**, **Exit**, **Save**, etc...) can be selected with the :arrow_left: arrow left and :arrow_right: arrow right keys.

The kernel configuration options with a left **[*]** symbol are marked to be built, while the ones with a left **&lt;M&gt;** symbol are marked to be built as modules; the kernel configuration options with a left **[ ]** symbol are unmarked and excluded from the compilation.

In order to exclude a field from the compilation, you have to reach it and press the **n** key.

The fields with a right **---&gt;** symbol are submenus, which can be accessed by typing the :leftwards_arrow_with_hook: enter key with the **Select** action highlighted.

In order to return to the parent menu (if you are in a submenu), or exit (if you already are in the main menu), you can select the action **Exit** and press the :leftwards_arrow_with_hook: enter key.

In order to save the configuration, you can select the action **Save** and press the :leftwards_arrow_with_hook: enter key.

Following the instructions above, we can completely remove the network interfaces by unmarking (by the **n** key) the following fields:
- Networking Support ---&gt;&nbsp;&nbsp;Wireless ---&gt;&nbsp;&nbsp;**cfg80211 – wireless configuration API**
- Device Drivers ---&gt;&nbsp;&nbsp;**Network devide support ---&gt;**<br/>This is a submenu, so you have to access it and then unmark every field in it and in its submenus; don't worry if a few fields stay marked, because they cannot be unmarked but won't invalidate the result.

After this, you can **Save** the new configuration and **Exit**.

Ubuntu (and Debian) adds a signature checking system directly to the kernel, then in order to correctly compile our custom kernel, we have to disable this feature by the following commands:

```
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```

## Kernel compilation
We can start the kernel compilation by the following command:

```
make -j $(nproc)
```

Now it's time to have a break and relax, since the kernel compilation will last for a long time.

:bulb: **TIP**: the _-j $(nproc)_ option attemts a parallel building based on the number of processors (saving some time).

:warning: **WARNING: if the compilation ends with some error logged, don't proceed with the next steps!**

## Kernel installation
When the compilation ends, we can install the new kernel by the commands:

```
sudo make modules_install
sudo make install
```

And then update the boot loader configuration:

```
sudo update-initramfs -c -k 5.19.17
sudo update-grub
```

![screenshot](/screenshots/grub_update.png?raw=true)

## Post installation actions
In order to apply the new kernel, firstly we have to reboot:

```
reboot
```

When the system goes up, we can open a terminal and check the kernel in use and the network status again:

```
uname -mrs
ip link
ip addr
```

If the only network interface present is the loopback (**lo**) interface, like in the picture below, then the procedure was successful! We can furtherly confirm it by a browser, or by opening the Electrum wallet and trying to connect it to any server... it will always stay offline!

![screenshot](/screenshots/network_status_end.png?raw=true)

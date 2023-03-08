# AirBuntu
How to build a custom kernel in order to keep your AirGap PC offline.

## Introduction
This tutorial shows how to build a custom kernel in order to remove any network interface.
Having an offline PC can be useful for example in order to implement a cold wallet (e.g. [Electrum](https://electrum.org)). Such goal could be achieved in different ways, e.g. by disabling the network manager, or blacklisting kernel modules; with this tutorial we will perform something stronger: instead of _disabling_ the ability to go online, we will completely _erase_ it.

This guide has been written and tested for Ubuntu, but il could be extended, with proper (and out of scope) modifications, to any linux distribution.

:warning: **WARNING**: the following actions are potentially destructive, if not properly done. We strongly suggest to use a fresh installation, or to backup all the important data before to proceed.


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

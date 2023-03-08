# airbuntu
How to build a custom kernel in order to keep your AirGap PC offline.

## Introduction
This tutorial shows how to build a custom kernel in order to remove any network interface.
Having an offline PC can be useful for example in order to implement a cold wallet (e.g. [Electrum](https://electrum.org)). Such goal could be achieved in different ways, e.g. by disabling the network manager, or blacklisting kernel modules; with this tutorial we will perform something stronger: instead of _disabling_ the ability to go online, we will completely _erase_ it.

This guide has been written and tested for Ubuntu (LTS 22.04), but il could be extended, with proper (and out of scope) modifications, to any linux distribution.

:warning: **WARNING**: the following actions are potentially destructive, if not properly done. We strongly suggest to use a fresh installation, or to backup all the important data before to proceed.


## Preparing the system


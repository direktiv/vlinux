# vlinux

This repository contains the changes to linux kernel. It containes patches for the different Linux versions for vorteil.io.

These patches are getting applied during the build phase of [vbundler](https://github.com/vorteil/vbundler) and are based on a simple naming convention: **_0001-vorteil-vX.X.X.patch_**

To use a different Linux version for a vorteil.io bundle change the following line in the Makefile of [vbundler](https://github.com/vorteil/vbundler):

```cmake
LINUX     := 'v5.8.8'
```

For more information read the documentation of [vbundler](https://github.com/vorteil/vbundler).

## Changes

The majority of changes are on module level and can be found in _drivers/_vorteil_:

**bootdev.c:** Creates the file _/proc/bootdev_ which stores the name of the root device (e.g. /dev/sda) which can be different for different cloud environments. It is consumed by [vinitd](https://github.com/vorteil/vinitd) during startup.

**disk.c:** Finds the root device based on a static PARTUUID used by [vorteil.io tools](https://github.com/vorteil/vorteil) and stores it for _/proc/bootdev_.

**vlogfsfs.c, vlogfs_ops.c:** Used for redirects of log files. It is based on the simple ram filesystem. Every write is getting redirected in a circular buffer. This data is read and redirected by [fluentbit](https://fluentbit.io/) included in vorteil.io virtual machines if logging is configured.

**vorteilfs.c:**  Simple tar filesystem serving all files in the bundle for that vorteil.io virtual machine. Contains at least Linux and [vinitd](https://github.com/vorteil/vinitd).

**vtty.c:** Manages the stdout of vorteil.io virtual machines. It can write to tty, ttyS and fluentbit in parallel. Configuration is via ioctls with valid values 0 (stdout, serial), 1 (stdout), 2 (serial), 3 (no output). The ioctl value 4 is independent from the default poutput and enables fluentbit redirects of stdout.  

Two additional changes have been made:

**init/main.c (function kernel_init):** Mounts the vorteil filesystem under _/vorteil_.

**init/do_mounts.c (function mount_root):** Changes the root mount from _/dev/root_ to the name of the actual roiot device, e.g. _/dev/sda2_.

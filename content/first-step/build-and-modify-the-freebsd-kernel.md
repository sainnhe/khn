---
title: 'Build and Modify the FreeBSD Kernel'
comments: true
breadcrumbs: false
weight: 3
sidebar:
  open: true
---

## Build and Run the FreeBSD Kernel

Compared to the Linux kernel, it's relatively easy to build and run the FreeBSD kernel.

### Get the Source Code

Typically, the source code is installed along with the operating system in `/usr/src`. If this directory doesn't exist, you can obtain the source code like this:

```
# git clone https://git.freebsd.org/src.git /usr/src
# cd /usr/src
# git checkout release/14.2.0
```

Nevertheless, if you have this directory, it's still recommended to clone the repository and move `.git` to `/usr/src` so we can perform version control conveniently.

```
# git clone https://git.freebsd.org/src.git ~/freebsd
# cd ~/freebsd
# git checkout release/14.2.0
# mv ~/freebsd/.git /usr/src
# cd /usr/src
# git status
```

The above operation will set the directory permissions to be writable only by the root user. If you want normal users to also have write permissions so we can develop in normal user, you can add the user to the `wheel` user group and set the directory's group to `wheel`.

```
# pw groupmod wheel -m your_username
# chgrp -R wheel /usr/src
# chmod g+s /usr/src
# find /usr/src -type d -exec chmod g+rwx {} \;
# find /usr/src -type f -exec chmod g+rw {} \;
```

### Build and Run the FreeBSD Kernel

> [!CAUTION]
> Before continue, be sure to back up the virtual machine to prevent it from being unable to start if the kernel is modified.

Let's first build and install a kernel that uses the default configuration:

```
$ su root
# cd /usr/src
# make buildkernel -j$(sysctl -n hw.ncpu)
# make installkernel
```

Where `sysctl -n hw.ncpu` returns the value of CPU cores.

After installation, reboot the VM and see if the kernel works.

To generate `compile_commands.json`, which provides compilation information for language servers, replace the first `make` commands with the following command:

```
# intercept-build --append make buildkernel -j$(sysctl -n hw.ncpu)
```

The `intercept-build` command is bundled with `llvm19` package, which intercepts build process to generate `compile_commands.json`. The `--append` flag tells the `intercept-build` to read an existing `compile_commands.json` if it exists and append the results to the database, also merge duplicated command keys.

The `buildkernel` target will use a default configuration named `GENERIC`, which supports a wide range of hardwares. This configuration file is located in `/usr/src/sys/amd64/conf/GENERIC` on x86\_64 machine [{{< icon "bookmark" >}} [freebsd] ch1: x86\_64 conf](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/sys/amd64/conf/GENERIC#L1), and `/usr/src/sys/arm64/conf/GENERIC` on arm64 machine [{{< icon "bookmark" >}} [freebsd] ch1: arm64 conf](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/sys/arm64/conf/GENERIC#L1).

To use your own customized kernel configuration, copy the generic configuration file and edit it:

```
# cd /usr/src/sys/amd64/conf
# cp GENERIC MYKERN
# vim MYKERN
```

The format of the kernel configuration file is simple. Each line contains a keyword that represents a device or subsystem, an argument, and a brief description. Any text after a `#` is considered a comment and ignored. For the specifications of the kernel configuration, refer to [config(5)](https://man.freebsd.org/cgi/man.cgi?query=config&sektion=5&format=html).

After editing your configuration file, save it and execute the following command to build the kernel:

```
# make buildkernel KERNCONF=MYKERN
```

## Modify the FreeBSD Kernel

Same as the Linux kernel, we are about to print a "Hello world!" string when the kernel starts.

The stuff similar to Linux's `arch/x86/kernel/head_64.S` [{{< icon "bookmark" >}} [linux] ch1: entry point x86\_64](https://github.com/torvalds/linux/blob/v6.12/arch/x86/kernel/head_64.S#L1) is `sys/amd64/amd64/locore.S` [{{< icon "bookmark" >}} [freebsd] ch1: entry point x86\_64](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/sys/amd64/amd64/locore.S#L1),
the stuff similar to Linux's `start_kernel()` [{{< icon "bookmark" >}} [linux] ch1: start_kernel()](https://github.com/torvalds/linux/blob/v6.12/init/main.c#L903) is `mi_startup()` defined in `sys/kern/init_main.c` [{{< icon "bookmark" >}} [freebsd] ch1: mi_startup()](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/sys/kern/init_main.c#L260),
and the stuff similar to Linux's `printk()` [{{< icon "bookmark" >}} [linux] ch1: printk()](https://github.com/torvalds/linux/blob/v6.12/include/linux/printk.h#L490) is `printf()` defined in `sys/sys/systm.h` [{{< icon "bookmark" >}} [freebsd] ch1: printf()](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/sys/sys/systm.h#L223).

So all we need to do is adding a `printf()` to `mi_startup()` that prints "Hello world!" string. The patch is as follows:

```diff {filename="patch.diff"}
diff --git a/sys/kern/init_main.c b/sys/kern/init_main.c
index 1575287716ee..4523b912d6be 100644
--- a/sys/kern/init_main.c
+++ b/sys/kern/init_main.c
@@ -334,6 +334,7 @@ mi_startup(void)
 		/* Check off the one we're just done */
 		last = sip->subsystem;
 	}
+	printf("Hello world!\n")
 
 	TSEXIT();	/* Here so we don't overlap with start_init. */
 	BOOTTRACE("mi_startup done");
```

Apply this patch, rebuild and install the kernel, reboot the VM and you can see the effect in `dmesg`:

```shell
$ dmesg | grep Hello
Hello world!
```

## More Resources

- [Chapter 10. Configuring the FreeBSD Kernel](https://docs.freebsd.org/en/books/handbook/kernelconfig/)
- [Chapter 1. Bootstrapping and Kernel Initialization](https://docs.freebsd.org/en/books/arch-handbook/boot/)

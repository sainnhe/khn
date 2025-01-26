---
title: 'Build and Modify the Linux Kernel'
comments: true
breadcrumbs: false
weight: 2
sidebar:
  open: true
---

Now that we have completed the first thing mentioned in the [First Step](/first-step), which is setup your development environment. There are two more things to do:

1. Build and run the kernel.
2. Modify at least one line of code and see the effect of the modification.

Let's get started.

## Build and Run the Linux Kernel

### Build the Linux Kernel

#### Get the Source Code

The very first question on how to build the Linux kernel is where to get the source code. If you google it, you may find a GitHub repository: [https://github.com/torvalds/linux](https://github.com/torvalds/linux)

However this is simply a mirror of another upstream repository hosted on kernel.org, whose URL is [https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git)

This repository is also known as the **mainline** repository, and is maintained by Linus Torvalds. It contains the cutting-edge code and updates very frequently, usually with new commits every day. These commits may include new features, performance improvements, bugfixes, and more.

When the code in the mainline repository is deemed stable enough, Torvalds will create a tag such as v6.12 and then officially announce a release. These releases are known as stable releases.

So what will happen if there are new bugs or security flaws found in a stable release? Well, another group of maintainers called "stable kernel maintainers" is responsible for applying patches backported from the mainline tree, and they will tag bugfix releases like v6.12.3 and v6.12.4. They do so in another repository, aka **stable** repository: [https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git)

Each stable release will receive bugfix and security updates for a period of time in stable repository, usually until the next stable release is available. However, if a stable release becomes a LTS (Long-term Support) release, it'll receive longer support, generally for several years.

Releases in stable repository is considered stable enough for use in production, so most of the Linux distros choose to package Linux kernel from the stable repository, leaving mainline repository for development purpose.

In this book, we'll focus on the latest LTS release at the time of writing, i.e. `v6.12`. This tag exists in both mainline repository and stable repository, so you can clone one of them and checkout this tag. You may notice that there are many bugfix releases of v6.12 in stable repository, but since bugfixes are not our focus, we mainly focus on the mainline stable release v6.12. If you are curious about these bugfixes, you can use git diff to compare and see the differences between various bugfix releases.

```shell
$ git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd linux
$ git checkout v6.12
```

#### Build the Kernel

> [!NOTE]
> You may need to install some packages to build the kernel. If you are using Arch Linux and some commands are missing, you can use `pacman -F` to search for corresponding package that contains the given binary. For users of other Linux distros, you can find the package name of a missing command in [command-not-found.com](https://command-not-found.com/).

First, we need to generate kernel configuration. The kernel configuration determines which features and drivers are included in the final kernel image. Type the following command to see all available configuration targets:

```shell
$ make help
```

The `defconfig` target will generate a default config based on your current architecture. So let's get started by generating a default config:

```shell
$ make defconfig
```

The generated file is named `.config` and is located in the project root directory. You can use your editor to open this file and you'll see tons of configuration options.

Of course you can use your editor to edit these configuration options, but a better approach is to use a TUI menu to modify these options so we can have a clearer overview of all these configuration options:

```shell
$ make menuconfig
```

All the modifications will be saved in `.config`. The next question is how to reproduce your modified config in another machine? One way is directly copying `.config` to another machine, but there is a problem with this approach, that is, if the architecture of another machine is different from the current machine, then the `.config` file may not be applicable to the other machine.

Can we save the differences between the modified config and the default config, and apply the differences in another machine's default config? In this way, the config used in another machine can be based on the default config of another machine's architecture. To do this, type:

```shell
$ make savedefconfig
```

This command will generate a file `defconfig` in the project root directory which contains minimal configuration that includes only the symbols that differ from the default configuration. Copy this file to another machine and you can reproduce the config by executing `make defconfig`.

Another question is what if you pull from the remote repository and some configuration options have changed? To update `.config` based on current kernel source code, execute:

```shell
$ make oldconfig
```

`make help` will print not only configuration targets, but also configuration topic targets which can easily enable a feature without having to modify dozens of configuration options. For example, to enable KVM (Kernel-based Virtual Machine) guest support:

```shell
$ make kvm_guest.config
```

After generating the configuration file, you can use the following command to compile the kernel:

```shell
$ make -j4
```

The `-j4` option tells `make` to run 4 jobs simultaneously. If you omit this option, `make` will only run 1 job, which may not maximize your CPU utilization. If you want to use the exact CPU core number on your machine as the job number, you can use `$(nproc)` to get the number of logical cores:

```shell
$ make -j$(nproc)
```

The output kernel image is located in `arch/x86_64/boot/bzImage` in x86\_64 machine and `arch/arm64/boot/Image` in arm64 machine.

The default compiler toolchain used in config file is gcc, but you can also use clang to build the kernel, and all you need to do is simply appending `CC=clang` to `make` commands. Here, since I use clangd in my code editor, I'll use clang to build the Linux kernel to keep the toolchain consistent. The full commands are listed below:

```shell
$ make CC=clang defconfig
$ make CC=clang -j$(nproc)
$ ./scripts/clang-tools/gen_compile_commands.py
```

The last command is used to generate `compile_commands.json`, which provides compilation information for clangd and ccls, as mentioned in previous article. When you open your code editor, clangd / ccls will automatically read this file and parse the project based on it.

### Run the Linux Kernel

Now let's run our kernel.

When the system boots, the Linux kernel takes control and initializes hardware components, mounts the root file system, and executes a user-space program, which by default is `/init`.

So to boot the kernel, we need to prepare a root file system, and a `/init` script.

Many articles on other websites will suggest you to compile and build a root file system, for example a busybox based root file system. But compiling these things manually will not only be troublesome, but also you can’t install softwares using package managers like in a Linux distro because busybox doesn't have a package manager.

So, why not directly use a root file system of a Linux distro?

Here, we choose to use Alpine Linux's rootfs because it's very small.

Go to the official download page of Alpine Linux: [https://alpinelinux.org/downloads/](https://alpinelinux.org/downloads/)

In the "Mini root filesystem" section, find and download the tarball that matches your CPU architecture.

Then in your terminal, switch to the root user to ensure the privilege is correct, and execute the following commands:

```
$ su root
# mkdir rootfs && cd rootfs
# tar xvf /path/to/alpine-minirootfs-<version>-<arch>.tar.gz
# vim init
```

Insert the following content to `/init`:

```shell {filename="/init"}
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys

ifconfig eth0 up
udhcpc eth0

mkdir -p /mnt/shared
mount -t 9p -o trans=virtio shared_mount /mnt/shared

exec /bin/sh
```

In this script, we mounted 2 virtual file system to `/proc` and `/sys`, brings up the network interface `eth0` and uses udhcpc client to obtain an IP address via DHCP. We also mounted a shared directory to `/mnt/shared` so the guest machine can access shared files from the host machine via this directory.

Save this file and execute the following commands:

```
# chmod +x init
# echo "nameserver 8.8.8.8" > etc/resolv.conf
# mkdir -p dev
# mknod -m 622 dev/console c 5 1
# mknod -m 666 dev/null c 1 3
# find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../initramfs.img && chmod a+r ../initramfs.img
```

We first make the init script executable, then we set the DNS resolver to google dns `8.8.8.8`. We also make two character devices for console and null output. Finally, we use `find`, `cpio` and `gzip` to generate a compressed cpio archive containing all the files in the current directory and its subdirectories. This archive is then saved as `initramfs.img` and made readable for all users.

Now, switch to normal user and execute the following command to launch a virtual machine:

{{< tabs items="x86_64,arm64" >}}

  {{< tab >}}

  ```shell
  $ mkdir shared
  $ qemu-system-x86_64 \
      -kernel /path/to/linux/arch/x86_64/boot/bzImage \
      -initrd /path/to/initramfs.img \
      -append "console=ttyS0" \
      -nographic \
      -m 500M \
      -netdev user,id=net0 \
      -device e1000,netdev=net0 \
      -fsdev local,id=shared_dir,path=./shared,security_model=none \
      -device virtio-9p-pci,fsdev=shared_dir,mount_tag=shared_mount
  ```

  In these commands:

  - We first create a directory named `shared`. This directory will be mounted in the guest virtual machine so the host machine can share files with the guest via this directory.
  - `-kernel` specifies the path of the kernel image.
  - `-initrd` specifies the path of the initial RAM filesystem (initramfs) image.
  - `-append "console=ttyS0"` appends a parameter to the kernel. Here, `console=ttyS0` directs the kernel's console output to the first serial port (ttyS0).
  - `-nographic` tells QEMU to run without a graphical window. Instead, it redirects the virtual machine's graphics output to the terminal.
  - `-m 500M` sets the amount of memory allocated to the virtual machine.
  - `-netdev user,id=net0` configures a user-mode network stack for the virtual machine. It creates a virtual network device (net0) that allows the VM to communicate with the host and external networks through NAT (Network Address Translation).
  - `-device e1000,netdev=net0` attaches an Intel e1000 network adapter to the virtual machine and connects it to the previously defined network device (net0). This enables networking within the VM.
  - `-fsdev local,id=shared_dir,path=./shared,security_model=none` sets up a file system device that allows sharing a directory from the host machine to the guest virtual machine. Here, it shares the `./shared` directory on the host with the guest.
  - `-device virtio-9p-pci,fsdev=shared_dir,mount_tag=shared_mount` attaches a virtio 9p file system device to the virtual machine, linking it to the previously defined file system device (shared\_dir).

  {{< /tab >}}

  {{< tab >}}

  ```shell
  $ mkdir shared
  $ qemu-system-aarch64 \
      -M virt \
      -cpu cortex-a57 \
      -kernel /path/to/linux/arch/arm64/boot/Image \
      -initrd /path/to/initramfs.img \
      -serial stdio \
      -m 500M \
      -netdev user,id=net0 \
      -device e1000,netdev=net0 \
      -fsdev local,id=shared_dir,path=./shared,security_model=none \
      -device virtio-9p-pci,fsdev=shared_dir,mount_tag=shared_mount
  ```

  In these commands:

  - We first create a directory named `shared`. This directory will be mounted in the guest virtual machine so the host machine can share files with the guest via this directory.
  - `-M virt` specifies the machine type for the virtual machine. "virt" refers to a generic, versatile platform that is commonly used for testing and development purposes on ARM architectures.
  - `-cpu cortex-a57` sets the CPU type for the virtual machine to Cortex-A57.
  - `-kernel` specifies the path of the kernel image.
  - `-initrd` specifies the path of the initial RAM filesystem (initramfs) image.
  - `-serial stdio` redirects the serial console output to the standard input/output of the terminal where QEMU is run, so we can interact with the virtual machine's console directly from the terminal.
  - `-m 500M` sets the amount of memory allocated to the virtual machine.
  - `-netdev user,id=net0` configures a user-mode network stack for the virtual machine. It creates a virtual network device (net0) that allows the VM to communicate with the host and external networks through NAT (Network Address Translation).
  - `-device e1000,netdev=net0` attaches an Intel e1000 network adapter to the virtual machine and connects it to the previously defined network device (net0). This enables networking within the VM.
  - `-fsdev local,id=shared_dir,path=./shared,security_model=none` sets up a file system device that allows sharing a directory from the host machine to the guest virtual machine. Here, it shares the `./shared` directory on the host with the guest.
  - `-device virtio-9p-pci,fsdev=shared_dir,mount_tag=shared_mount` attaches a virtio 9p file system device to the virtual machine, linking it to the previously defined file system device (shared\_dir).

  {{< /tab >}}

{{< /tabs >}}

If you compiled the kernel with KVM guest supported, you can also append `-enable-kvm` to enable KVM.

Now you successfully boot a virtual machine with the kernel compiled by yourself, and has access to the Internet and can share files with host machine.

Let's try to install vim in your virtual machine:

```
# apk add vim
```

## Modify the Linux Kernel

Now there is only one last thing left, which is to modify at least one line of code and see the effect of the modification. In this section, we'll modify the Linux kernel and let it print a "Hello world!" string when the kernel starts.

We first need to find out the entry point of the Linux kernel. When we write a C program, we use `main()` function as the entry point. In the Linux kernel however, there is no `main()` function, instead the kernel execution starts with architecture-specific assembly code. For x86\_64, it's `arch/x86/kernel/head_64.S` [{{< icon "bookmark" >}} [linux] ch1: entry point x86\_64](https://github.com/torvalds/linux/blob/v6.12/arch/x86/kernel/head_64.S#L1), and for arm64, it's `arch/arm64/kernel/head.S` [{{< icon "bookmark" >}} [linux] ch1: entry point arm64](https://github.com/torvalds/linux/blob/v6.12/arch/arm64/kernel/head.S#L1).

This assembly code handles very low-level tasks. It first sets up the CPU, then initializes a minimal stack, and prepares the environment for C code to run. Once the assembly code completes its job, it jumps to the `start_kernel()` function in `init/main.c` [{{< icon "bookmark" >}} [linux] ch1: start_kernel()](https://github.com/torvalds/linux/blob/v6.12/init/main.c#L903). This is where the "generic" kernel initialization begins in C.

Why no `main()` function in the Linux kernel? Remember that the kernel is not a userspace program and does not rely on the standard C runtime (which typically provides `main()`). This also means *you cannot use standard libraries* in the kernel source code, like `printf()` in `<stdio.h>` and `strcpy()` in `<string.h>`.

But don't be afraid, although we cannot use standard libraries, the kernel implements a set of libraries that can be used to replace most of the functions in standard libraries. For example, for `printf()` we have `printk()` defined in `<linux/printk.h>` [{{< icon "bookmark" >}} [linux] ch1: printk()](https://github.com/torvalds/linux/blob/v6.12/include/linux/printk.h#L490), and for `strcpy()` we have `strscpy()` defined in `<linux/string.h>` [{{< icon "bookmark" >}} [linux] ch1: strscpy()](https://github.com/torvalds/linux/blob/v6.12/include/linux/string.h#L112).

We can use `printk()` like this:

```c
printk(KERN_INFO "Hello world! Value: %d\n", 42);
```

Where `KERN_INFO` is the log level defined in `<linux/kern_levels.h>` [{{< icon "bookmark" >}} [linux] ch1: KERN_INFO](https://github.com/torvalds/linux/blob/v6.12/include/linux/kern_levels.h#L14).

`<linux/printk.h>` also defines some simple wrappers for `printk()`, for example the above code is equivalent to `pr_info("Hello world! Value: %d\n", 42)`. `pr_info()` is a macro that wraps `printk(KERN_INFO ...)` [{{< icon "bookmark" >}} [linux] ch1: pr_info()](https://github.com/torvalds/linux/blob/v6.12/include/linux/printk.h#L562).

Now let's come back to our question. How to print a "Hello world!" string when the kernel starts? Through the above analysis we know that we can modify the `start_kernel()` function in `init/main.c` and add a line that uses `printk()` or its wrappers to print a string.

An example implementation is as follows. You can use `git apply` to apply the patch, then re-execute `make` to compile the kernel, and then use qemu to start the kernel to see the effect.

```diff {filename="patch.diff"}
diff --git a/init/main.c b/init/main.c
index c4778edae797..bbc6a44ed5bd 100644
--- a/init/main.c
+++ b/init/main.c
@@ -936,6 +936,7 @@ void start_kernel(void)
 	boot_cpu_hotplug_init();
 
 	pr_notice("Kernel command line: %s\n", saved_command_line);
+	pr_notice("Hello world!\n");
 	/* parameters may set static keys */
 	parse_early_param();
 	after_dashes = parse_args("Booting kernel",
```

## More Resources

- [Active kernel releases](https://www.kernel.org/category/releases.html)
- [Configuration targets and editors](https://docs.kernel.org/kbuild/kconfig.html)

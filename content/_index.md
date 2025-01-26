---
title: "Sainnhe's Kernel Hacking Notes"
comments: true
breadcrumbs: false
next: /first-step
cascade:
  type: docs
---

## Introduction

Learning the operating system kernel is hard, some even said that you shouldn't consider touching this area if you don't have experience of a decade of programming.

I don't think so. Programming experience should not be a barrier that prevents kernel enthusiasts from exploring this area, because developers will always learn and improve themselves in the process of doing projects and solving problems. And I found that the fastest way to improve myself is to read and hack the source code of the top open source projects.

This book is about how a kernel newbie tries to hack the kernel source code. I want to share what I've found in this process, and hope these notes can provide you some help.

Note that the kernel mentioned here does not specifically refer to the Linux kernel. Instead, I'm trying to hack some production-ready open source kernels. At this moment, this book will talk about both Linux and FreeBSD kernels.

You may wonder why to hack FreeBSD kernel, since Linux has already became a industry standard solution. Personally, I love FreeBSD because of it's stability and consistency of software packages, plus the hearsay of FreeBSD's clean codebase (and that's what I want to verify). Anyway, you are free to skip sections about FreeBSD if you're not interested.

## How to read this book

This book focuses on hands-on, which means that if you just read it without practice, you may not even understand what I'm saying.

And given the hands-on feature, this book will use a very different approach to discuss problems. This book will use code editors' bookmark plug-ins to mark the code mentioned in the book. You can search for the name of the bookmark in your code editor to quickly jump to the specified code position. Currently, Vim and VSCode is supported.

Each bookmark in this book will have a hyperlink pointing to the corresponding code snippet, so for non-Vim and VSCode users, you can also click on the link to view the mentioned code snippet. Besides, bookmarks adhere to the following naming convention:

[{{< icon "bookmark" >}} [\<kernel\>] \<annotation\>](https://www.example.com)

Where \<kernel\> is one of "linux" or "freebsd", and \<annotation\> is the string you should search for in your code editor's bookmark plug-in. For example, [{{< icon "bookmark" >}} [freebsd] ch1: README](https://github.com/freebsd/freebsd-src/blob/release/14.2.0/README.md#L1) means this bookmark is used to mark the code snippet in FreeBSD, and you should search for "ch1: README" in your code editor's bookmark plug-in.

In addition, since most modern editors support the function of jumping to definitions and references, when reading the code, you should try to jump to its definitions and references to understand how the corresponding interfaces are implemented and used.

When discussing some topics, I'll also attach some patches, and you can use [git apply](https://git-scm.com/docs/git-apply) to apply these patches and try them out.

At the end of each section, I will list some resources that I found useful. You can read these resources further to better understand the relevant topics.

In addition, you can use an RSS client to subscribe to updates of this book. The RSS subscription link can be found in the sidebar, and you can find many open source RSS clients on GitHub. To name a few:

- [FluentReader](https://github.com/yang991178/fluent-reader): Available on Windows, macOS, Linux, iOS and Android.
- [NetNewsWire](https://github.com/Ranchero-Software/NetNewsWire): Available on macOS and iOS.
- [ReadYou](https://github.com/Ashinch/ReadYou): Available on Android.

## Prerequisites

This book assumes you:

- Can write programs in the C programming language. I'd recommend[《C Primer Plus》](https://www.oreilly.com/library/view/c-primer-plus/9780133432398/)if you haven't learned it yet.
- Know how to use git. The [official book](https://git-scm.com/book/en/v2) is a good learning resource.
- Understand the basics of operating systems and networking and know how to perform UNIX system programming. I'd recommend[《The Linux Programming Interface》](https://man7.org/tlpi/)for both learning and reference.
- Be familiar to UNIX shell commands.[《Harley Hahn's Guide to Unix and Linux》](https://www.harley.com/unix-book/book/chapters/home.html)is a good guide if you haven't learned it systematically.
- Have a working Linux environment, whether it is a virtual machine, dual system, [WSL](https://learn.microsoft.com/en-us/windows/wsl/about) (Windows Subsystem for Linux), cloud server or a docker container. If you are using macOS, you can use [OrbStack](https://orbstack.dev/) or [Lima](https://github.com/lima-vm/lima) as an alternative to WSL.
- Have a working [FreeBSD 14.2.0](https://www.freebsd.org/where/) virtual machine, and a backup copy of this VM. We'll directly build and install the FreeBSD kernel in a VM, so make sure you have a backup of it. You don't necessarily need to set up a desktop environment, instead you can simply set up a sshd service and log in to this VM with ssh.

In this book, I'll use Arch Linux (x86\_64) as development environment. All the operations mentioned in this article are guaranteed to be reproducible on Arch Linux. Note that this does not include Arch Linux derivatives such as Manjaro and Arch Linux Arm. If you encounter any problems in the process of reproducing, please feel free to discuss in the comment area.

## Kernel source code versions

This book will discuss the latest LTS (Long-term support) versions of Linux and FreeBSD at the time of writing. Specifically, the corresponding git tags are as follows:

{{< cards >}}
  {{< card link="https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tag/?h=v6.12" title="Linux v6.12" icon="tag" >}}
  {{< card link="https://cgit.freebsd.org/src/tag/?h=release/14.2.0" title="FreeBSD release/14.2.0" icon="tag" >}}
{{< /cards >}}

## Credits

This book is open source, and licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). The source code can be found on GitHub: [https://github.com/sainnhe/khn](https://github.com/sainnhe/khn)

In addition, I would like to give special thanks to the following two open source projects:

1. [Hugo](https://gohugo.io/): This website uses hugo to generate static web pages.
2. [Hextra](https://imfing.github.io/hextra/): This website uses hextra as hugo theme template.

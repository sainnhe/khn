---
title: 'Setup Your Code Editor'
comments: true
breadcrumbs: false
prev: /first-step
weight: 1
sidebar:
  open: true
---

In this section, we will discuss how to setup a development environment using Vim or VSCode, both need to configure a language server and install a bookmark plug-in.

{{< callout type="warning" >}}
The following guide assumes the host is a working Linux operating system. If you are a VSCode user and the Linux operating system runs as a guest in a virtual machine or a container, you may need to install [VSCode Remote Development Suite](https://code.visualstudio.com/docs/remote/remote-overview) to allow VSCode to access the Linux environment.
{{< /callout >}}

## Language Server

A language server is a program that provides language-specific features like code completion, linting, and syntax highlighting to development environments through the Language Server Protocol. There are 2 actively maintained language servers of C/C++:

1. [clangd](https://clangd.llvm.org/): Maintained by [LLVM](https://llvm.org)
2. [ccls](https://github.com/MaskRay/ccls): Maintained by [Fangrui Song](https://github.com/MaskRay)

Both of them will read a file in the root of a project named `compile_commands.json`, which provides clangd and ccls compilation information so they can correctly analyze the source code. We'll talk about how to generate this file later.

Take clangd for example, there are 3 steps to get it working:

1. Install clangd on your machine: [Installing clangd](https://clangd.llvm.org/installation#installing-clangd)
2. Install code editor plug-in: [Editor plugins](https://clangd.llvm.org/installation#editor-plugins)
3. Configure the clangd plug-in to use the following arguments:

```sh
clangd \
  --background-index \
  --header-insertion=never
```

`--background-index` tells clangd to index project in the background and persist index on disk. This is very important because the kernel's codebase is very large, and it'll take a long time to index, so you generally don't want clangd to recreate index every time you open the project.

`--header-insertion=never` tells clangd not to insert headers on completion. We need this because clangd will falsely insert unnecessary headers.

The instructions on how to configure editors to use these 2 arguments can be found in [Editor plugins](https://clangd.llvm.org/installation#editor-plugins).

Additionally, execute `clangd --help` to have a look at all available arguments of clangd. Personally, I use the following arguments:

```sh
clangd \
  --background-index \
  --header-insertion=never \
  --clang-tidy \
  --completion-style=detailed
```

Alternatively, if you want to use ccls instead of clangd, follow the instructions on the project's wiki page: [ccls wiki](https://github.com/MaskRay/ccls/wiki)

## Bookmark

This book relies on editors' bookmark plug-in to mark mentioned code, so you need to install and setup the bookmark plug-in in your editor.

{{< tabs items="Vim,VSCode" >}}

  {{< tab >}}

  Install [MattesGroeger/vim-bookmarks](https://github.com/MattesGroeger/vim-bookmarks) and add `let g:bookmark_save_per_working_dir = 1` to your vimrc, so bookmarks will be stored in a file named `.vim-bookmarks` in the current working directory.

  Then you can download the bookmark files used by this book, and put them in the project root, which is the source code of the kernels that we'll obtain later. Reopen your code editor, and you can see the bookmarks.

  The download URLs are listed below:

  {{< cards >}}
    {{< card link="https://github.com/sainnhe/khn/blob/master/bookmarks/vim/linux.vim" title="Linux bookmarks for Vim" icon="bookmark" >}}
    {{< card link="https://github.com/sainnhe/khn/blob/master/bookmarks/vim/freebsd.vim" title="FreeBSD bookmarks for Vim" icon="bookmark" >}}
  {{< /cards >}}

  {{< /tab >}}

  {{< tab >}}

  Install [alefragnani.Bookmarks](https://marketplace.visualstudio.com/items?itemName=alefragnani.Bookmarks) and add `"bookmarks.saveBookmarksInProject": true` to your settings.json, so bookmarks will be stored in a file named `.vscode/bookmarks.json` in the current working directory.

  Then you can download the bookmark files used by this book, and put them in the project root, which is the source code of the kernels that we'll obtain later. Reopen your code editor, and you can see the bookmarks.

  The download URLs are listed below:

  {{< cards >}}
    {{< card link="https://github.com/sainnhe/khn/blob/master/bookmarks/vscode/linux.json" title="Linux bookmarks for VSCode" icon="bookmark" >}}
    {{< card link="https://github.com/sainnhe/khn/blob/master/bookmarks/vscode/freebsd.json" title="FreeBSD bookmarks for VSCode" icon="bookmark" >}}
  {{< /cards >}}

  {{< /tab >}}

{{< /tabs >}}


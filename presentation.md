---
author: 'Marc Koderer'
title: 'How to Contribute to the Linux Kernel'
patat:
    wrap: true
    theme:
        emph: [vividBlue, onVividBlack, italic]
        strong: [bold]
        imageTarget: [onDullWhite, vividRed]
        codeBlock: [vividGreen]
        code: [vividGreen]
        header: [rgb#f08000]
        syntaxHighlighting:
             decVal: [bold, onDullRed]
             comment: [rgb#DCDCDC]
    eval:
        figlet:
             command: figlet -f basic
             fragment: false
             replace: true
        cat:
             command: cat --
             fragment: false
             replace: true
...

```figlet
Agenda
```

* Build your dev environment
* Kernel dev structure
* How to contribute / communicate
* About testing
* Where to start..

---

# Build your dev environment

```figlet
Dev setup
```

---
# Build an environment

Vagrant with your picked distro

Dont forget to resize you volume::
```bash
    vagrant plugin install vagrant-disksize
```

- Don't forget to increase CPU, MEM and disk
- Do not use shared FS for the kernel git

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.disksize.size = '100GB'

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
    v.name = "kernel"
  end
end
```
---

## Install dependencies

```bash
   sudo apt-get update
   sudo apt-get install -y libncurses-dev gawk flex bison bc
   sudo apt-get install -y openssl libssl-dev dkms libelf-dev
   sudo apt-get install -y libudev-dev libpci-dev libiberty-dev autoconf
   sudo apt-get install -y git
   # Optional
   sudo apt install linux-headers-$(uname -r)
```

---

## Clone the sources
Take a coffee
```
> git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
> git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git

Cloning into 'linux'...
remote: Enumerating objects: 248, done.
remote: Counting objects: 100% (248/248), done.
remote: Compressing objects: 100% (148/148), done.
remote: Total 7990947 (delta 147), reused 140 (delta 100), pack-reused 7990699
Receiving objects: 100% (7990947/7990947), 2.12 GiB | 5.70 MiB/s, done.
Resolving deltas: 100% (6545813/6545813), done.
Updating files: 100% (71499/71499), done.
```
---

## Building it

```bash
# copy current config (or make your own one with make menuconfig)
cp /boot/config-`uname -r`* .config
make -j4
make modules
sudo make modules_install
sudo make install
sudo update-grub
```

---

## Changing something and prepre for a commit

```bash
git checkout -b first-patch
git add <file>
git commit -s -v
git format-patch -o /tmp/ HEAD^
```

---

# Kernel dev structure

```figlet
Structure
```

---

## Kernel Architecture

```cat
       ┌─────────────────────────────────────────────────────────────────┐
       │  Applications                                                   │
       │                           ┌────────────────────────────┐        │
       └───────────────────────────┘System Libs (e.g. libc)     └────────┘
                                        │
┌───────────┐ ┌─────────────────────────▼─────────────────────────────────────┐
│ Modules   │ │  ┌──────────────────────────────────────────────────────────┐ │
│           │ │  │ System Call Interface (SCI)                              │ │
│           │ │  │                                                          │ │
│           │ │  └──────────────────────────────────────────────────────────┘ │
│           │ │   ┌───────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│           │ │   │    I/O            │  │     Process     │  │ Virt / Cont │ │
│           │ │   │                   │  │                 │  │             │ │
│           │ │   │ - File System     │  │ - Scheduler     │  │ - KVM       │ │
│           │ │   │ - Networking      │  │ - Mem Mgmt      │  │ - Cgroups   │ │
│           │ │   │ - Device Drivers  │  │ - IPC           │  │ - Net NS    │ │
│           │ │   │                   │  │                 │  │             │ │
│           │ │   │                   │  │                 │  │             │ │
│           │ │   └───────────────────┘  └─────────────────┘  └─────────────┘ │
│           │ │  ┌──────────────────────────────────────────────────────────┐ │
│           │ │  │                                                          │ │
│           │ │  │  Arch dependent Code                                     │ │
│           │ │  │                                                          │ │
│           │ │  └──────────────────────────────────────────────────────────┘ │
└───────────┘ └───────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────────────┐
│                                   Hardware                                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Kernel Source - where to start

- `Makefile` This file is the top-level Makefile for the whole source tree
- `MAINTAINERS` Defines maintainers for each subsystem and area
- `Documentation/` This directory contains a lot of useful information about configuring the kernel
- `arch/`: All the architecture specific code is in this directory
- `crypto/` This is a cryptographic API for use by the kernel itself.
- `drivers/` As a general rule, code to run peripheral devices is found in subdirectories of this directory.
- `fs/` Both the generic filesystem code (VFS) and the code for each different filesystem
- `include/`: Most of the header files included at the beginning of a .c file
- `ipc/`: "IPC" stands for "Inter-Process Communication".
- `kernel/`: Generic kernel level code that doesn't fit anywhere else goes in here.
- `lib/`: Routines of generic usefulness to all kernel code are put in here.
- `net/`: The high-level networking code is here.
- `scripts/`: This directory contains scripts that are useful in building the kernel
- `security/`: Code for different Linux security models can be found here
- `usr/`: This directory contains code that builds a cpio-format

Source: https://courses.linuxchix.org/kernel-hacking-2002/08-overview-kernel-source.html

---

# How to contribute / communicate

```figlet
Let's get started
```

---
## Kernel commit structure
```cat
         ┌───────────────────────┐   ┌───────────────────────┐
         │ mainline Kernel       │   │ -next                 │
         │                       │   │                       │
         │  Linus Torvalds       │   │  Stephen Rothwell     │
         │                       │   │                       │
         └───────────────────────▲   ├───────────────────▲───┘
                 ▲           ▲   └───┴───────────────────┤
                 │           │                           │
┌────────────────┴──────┐ ┌──┴────────────────────┐ ┌────┴──────────────────┐
│ sub system            │ │ sub system            │ │ sub system            │
│                       │ │                       │ │                       │
│ Maintainer / ML       │ │ Maintainer / ML       │ │ Maintainer / ML       │
│                       │ │                       │ │                       │
└──────▲────────────────┘ └───────▲───────────────┘ └───────▲───────────────┘
       │                          │                         │
       │                          │                         │
 ┌─────┴────┐                ┌────┴─────┐              ┌────┴─────┐
 │Developer │                │Developer │              │Developer │
 └──────────┘                └──────────┘              └──────────┘
```

---


## Patch lifecycle

1. https://lkml.org/lkml/2019/8/6/510
2. https://lkml.org/lkml/2019/8/7/772
3. https://lore.kernel.org/patchwork/patch/1128028/
4. https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3ee8d6c592dc7fb240574b84e9f9a7f9db4d4b42


---

# Links

- [Linux Kernel Maintainacers](https://www.kernel.org/doc/html/latest/process/maintainers.html#maintainers-list)
- [How to configure GIT](https://www.kernel.org/doc/html/latest/maintainer/configure-git.html)
- [Source structure](https://courses.linuxchix.org/kernel-hacking-2002/08-overview-kernel-source.html)
- [Compile the Kernel](https://www.cyberciti.biz/tips/compiling-linux-kernel-26.html)
- [Git setup](https://opensource.com/article/18/8/first-linux-kernel-patch)

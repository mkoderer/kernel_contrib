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
* Do some changes and contribute it
* About testing
* Links

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


## Active Kernel Releases

[Kernel releases](https://www.kernel.org/category/releases.html)

- __Release Candidate (RC)__:
Mainline pre-releases used for testing and feature development. An RC a two weeks cycle.
- __Mainline__:
After muliple RC cycles a new mainline Kernel will be declared by Linus (2-3 months cycle)
- __Stable__:
Only bug fixes allowed and those will be backproted from mainline
- __Longterm (LTR)__:
Longterm maintenance Kernels only with imporant bug fixes/backports

---

## Kernel Source - where to start

- __Makefile__ This file is the top-level Makefile for the whole source tree
- __MAINTAINERS__ Defines maintainers for each subsystem and area
- __Documentation/__ This directory contains a lot of useful information about configuring the kernel
- __arch/__: All the architecture specific code is in this directory
- __crypto/__ This is a cryptographic API for use by the kernel itself.
- __drivers/__ As a general rule, code to run peripheral devices is found in subdirectories of this directory.
- __fs/__ Both the generic filesystem code (VFS) and the code for each different filesystem
- __include/__: Most of the header files included at the beginning of a .c file
- __ipc/__: "IPC" stands for "Inter-Process Communication".
- __kernel/__: Generic kernel level code that doesn't fit anywhere else goes in here.
- __lib/__: Routines of generic usefulness to all kernel code are put in here.
- __net/__: The high-level networking code is here.
- __scripts/__: This directory contains scripts that are useful in building the kernel
- __security/__: Code for different Linux security models can be found here
- __usr/__: This directory contains code that builds a cpio-format

Source: https://courses.linuxchix.org/kernel-hacking-2002/08-overview-kernel-source.html

---


# Do some changes and contribute it

```figlet
Let's get started
```

---

# Commit workflow

0. Create a branch `git chechout -b my_feature`
1. Do the change
2. Commit it `git add ..` and `git commit -s`
3. git format-patch -1
4. Check the patch scripts/checkpatch.pl my.patch
5. Find the right DL/maintainer with `scripts/get_maintainer.pl` 
6. Send out the contribution `git send-email`

---

# Practical example [1/4]

Compile `linux/samples/kprobes` load the modules and have a look:

```bash
make -C ~/kernel/linux/ M=$PWD modules
sudo insmod kprobe_example.ko
sudo dmesg | tail
  [  911.102788] <kernel_clone> post_handler: p->addr = 0x00000000cdf2e666, flags = 0x206
  [  913.116030] <kernel_clone> pre_handler: p->addr = 0x00000000cdf2e666, ip = ffffffffad87d011, flags = 0x206
  ..
```

Unload the kernel module
```bash
sudo rmmod kprobe_example.ko
sudo dmesg | tail -1
   [  919.433074] kprobe at 00000000cdf2e666 unregistered
```

In `kprobe_example.c` the probed entry was changed from `_do_fork` to `kernel_clone`.
This could cause issues when executing the sample in older kernels (kernel_clone was introduced quite recently).

---
# Practical example [2/4]

Let's change the code and probe a function that doesn't exist

```diff
diff --git a/samples/kprobes/kprobe_example.c b/samples/kprobes/kprobe_example.c
index 331dcf151532..794527f11667 100644
--- a/samples/kprobes/kprobe_example.c
+++ b/samples/kprobes/kprobe_example.c
@@ -15,7 +15,7 @@
 #include <linux/kprobes.h>

 #define MAX_SYMBOL_LEN 64
-static char symbol[MAX_SYMBOL_LEN] = "kernel_clone";
+static char symbol[MAX_SYMBOL_LEN] = "kernel_clone2";
 module_param_string(symbol, symbol, sizeof(symbol), 0644);

 /* For each probe you need to allocate a kprobe structure */
```

Build it and load it
```bash
make -C ~/kernel/linux/ M=$PWD modules
sudo insmod kprobe_example.ko
  insmod: ERROR: could not insert module kprobe_example.ko: Unknown symbol in module
sudo dmesg | tail -1
  [ 2143.150210] register_kprobe failed, returned -2
```

---

# Practical example [3/4]

The -2 means `ENOENT` `No such file or directory` [code](https://elixir.bootlin.com/linux/latest/source/include/uapi/asm-generic/errno-base.h#L6). Let's add a more obvious error message.

```diff
diff --git a/samples/kprobes/kprobe_example.c b/samples/kprobes/kprobe_example.c
index 331dcf151532..a85304890374 100644
-- a/samples/kprobes/kprobe_example.c
++ b/samples/kprobes/kprobe_example.c
@@ -108,7 +108,12 @@ static int __init kprobe_init(void)
        kp.fault_handler = handler_fault;

        ret = register_kprobe(&kp);
-       if (ret < 0) {
+       if (ret == -ENOENT){
+               /* Check in /proc/kallsyms for a valid symbol. */
+               pr_err("register_kprobe failed, symbol not found: %d\n", ret);
+               return ret;
+       }
+       else if (ret < 0) {
                pr_err("register_kprobe failed, returned %d\n", ret);
                return ret;
        }
```

---

# Practical example [4/4]

1. Let's extract a patch file

```bash
git checkout kprobe_err_msg
git format-patch -1
```

2. Check the patch
```bash
scripts/checkpatch.pl 0001-samples-kprobes-Adapt-error-handling.patch
```

3. Get the maintainers for the patch
```bash
scripts/get_maintainer.pl --no-rolestats 0001-samples-kprobes-Adapt-error-handling.patch
```

---


## Kernel commit structure
```cat
         ┌───────────────────────┐   ┌───────────────────────┐
         │ mainline Kernel       │   │ -next                 │
         │                       │   │                       │
         │  Linus Torvalds       │   │  Stephen Rothwell     │
         │                       │   │                       │
         └───────────────────────▲   └───────────────────▲───┘
                 ▲           ▲   └───────────────────────┤
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
# Contributing a patch
```bash
sudo apt-get install git-email
```

- content-type text/plain
- No HTML mails!
- No signatures
- No attachments
- Patches with git format-patch
- Patches with `git commit -s`

```gitconfig
[sendemail]
; setup for using git send-email; prompts for password
smtpuser = marc@koderer.com
smtpserver = SMTPSERVER
smtpencryption = tls
smtpserverport = 587
[user]
	email = marc@koderer.com
	name = Marc Koderer
```

---

# Links
- [Linux Foundation training](https://training.linuxfoundation.org/training/a-beginners-guide-to-linux-kernel-development-lfd103/)
- [Git flags](https://www.kernel.org/doc/html/latest/process/submitting-patches.html#using-reported-by-tested-by-reviewed-by-suggested-by-and-fixes)
- [Linux Kernel Maintainacers](https://www.kernel.org/doc/html/latest/process/maintainers.html#maintainers-list)
- [How to configure GIT](https://www.kernel.org/doc/html/latest/maintainer/configure-git.html)
- [Source structure](https://courses.linuxchix.org/kernel-hacking-2002/08-overview-kernel-source.html)
- [Compile the Kernel](https://www.cyberciti.biz/tips/compiling-linux-kernel-26.html)
- [Git setup](https://opensource.com/article/18/8/first-linux-kernel-patch)

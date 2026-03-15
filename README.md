[![Badge](https://img.shields.io/badge/Powered--by-NeuroSama-brightgreen?style=for-the-badge)](https://www.genome.gov/)

> ℹ️ **Warning**
> I am still working on to make the document better to read but it should work.
> 


# Compile the Linux kernel with boot logo support


[<img src="https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png" align="right" width="250">](https://www.archlinux.org/)


<p>This project is a personal experiment in compiling and installing a custom Linux kernel that displays the Tux logo during boot. The work was carried out on Arch Linux on March 8th, 2026.</p>

<p>This write‑up is still a work in progress and will be revised and expanded over time.</p>

<p>Below you can find the system configuration as it appeared before starting the experiment.</p>

**One important note:** I am using systemd‑boot as my bootloader. If you are using a different bootloader, you may need to adjust the installation steps based on your own research.
<br />
<br />
<br />



<img alt="20260308_13h33m57s_grim" src="https://github.com/user-attachments/assets/06ec15ef-31aa-4f7c-a8b9-0255c70ef44f" />

<br /><br />


Before we begin, we need to install the required dependencies to ensure the kernel can be compiled successfully.

```bash
sudo pacman -Syu
sudo pacman -S base-devel git ncurses flex bison openssl elfutils bc cpio
```

Once the dependencies are installed, the next step is to download the Linux source code and extract it to a working directory.

```bash
mkdir -p ~/kernel && cd ~/kernel
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.19.6.tar.xz
tar xf linux-6.19.6.tar.xz
cd linux-6.19.6
```

After extracting the source, copy the configuration file from your currently running kernel into the new kernel’s build directory. This ensures that your custom kernel starts with the same configuration as your existing one.

```bash
cp /usr/lib/modules/$(uname -r)/build/.config .config
make oldconfig

  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  LEX     scripts/kconfig/lexer.lex.c
  YACC    scripts/kconfig/parser.tab.[ch]
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
```

## Setting the kernel name and enable the tux logo

```bash
uname -a
Linux tux 6.19.6-arch1-1 #1 SMP PREEMPT_DYNAMIC Wed, 04 Mar 2026 18:25:08 +0000 x86_64 GNU/Linux
```

If you look at the last part of my kernel you notice it is named arch1-1. I want to change my own kernel to say -custom-tux at the end so lets do this. 

```bash
nake menuconfig
```

In the build menu go go **General setup** &rarr;  **Local version** and enter the custom name of the kernel.

<img alt="20260308_14h39m35s_grim" src="https://github.com/user-attachments/assets/44695eeb-4bd0-4627-b299-5b7ff6be320d" />

<br>Now use Exit to navigate back to the root of the menu (you will know you reached it if you see General Setup on top). Next navigate to **Device Drivers** &rarr; **Graphics support** to enable the **Bootup logo** option. You will see a page looking like this.
<br><img  alt="20260308_14h52m11s_grim" src="https://github.com/user-attachments/assets/d6c03918-6cb9-49d7-998d-fad9bd217a93" />

<br>Now still in the **Graphics support** menu gO into the **Frame buffer Devices**   &rarr; **Support for frame buffer device drivers" and make sure the followuing options are enabled.

- Vesa VGA graphics support
- EFI-based Framebuffer support

Next exit out to the main menu and dont forget to save the config on the way out. Before building the kernel validate your config by running the following command.

```bash
grep -E "LOGO|FB_SIMPLE|FB_EFI" .config

CONFIG_SYSFB_SIMPLEFB=y
CONFIG_FB_EFI=y
CONFIG_LOGO=y
CONFIG_LOGO_LINUX_MONO=y
CONFIG_LOGO_LINUX_VGA16=y
CONFIG_LOGO_LINUX_CLUT224=y
```

Next it is time to start building the kernel, be aware of the fact that depending on your system this might talke a while on my machine it will take. 


```bash
time make -j$(nproc)
```


> ℹ️ **Info**
> Compiling the Linux kernel on my machine took 20 minutes and 40 seconds. 
> 

If you dont want to messure the compile time use

```bash
make -j$(nproc)
```

Be aware compiling the kernel can take some time so take a coffee and watch some Nauro-sama videos voor like 20 minutes, your kernel will be waiting for you in about 20-30 minutes depending on your hardware.

```
sudo make modules_install
sudo make install
```
<br>

# Preparing for boot (with systemd boot)

**Systemd boot** after running make install with automatiscally create an boot entry for your freshly compiled kernel. But to be sure that you will actually see tux we need to make some changes to the entry file. If you look at the enties folder **/boot/loader/entries** you will see the newly created file.

```bash
ls /boot/loader/entries                  
 arch.conf   b473cb59c64b4556949412573488670b-6.19.6-custom-tux.conf
```

This is the boot entry the i have of my custom kernel. There are a few things to not here one i haved added **mitigations=off** to make my pc faster (this will disable cpu exploit protections so we warned) and i made sure that i removed **quiet** and **loglevel=3** but i added **fbcon=nodefer** because without this option tux was not visible at boot.


```bash
cat /boot/loader/entries/b473cb59c64b4556949412573488670b-6.19.6-custom-tux.conf
# Boot Loader Specification type#1 entry
# File created by /usr/lib/kernel/install.d/90-loaderentry.install (systemd 259.3-1-arch)
title      Arch Linux
version    6.19.6-custom-tux
machine-id b473cb59c64b4556949412573488670b
sort-key   arch
options    root=PARTUUID=616f35a6-cf84-45cd-a918-657014af5b61 zswap.enabled=0 rw rootfstype=ext4 mitigations=off systemd.machine_id=b473cb59c64b4556949412573488670b fbcon=nodefer
linux      /b473cb59c64b4556949412573488670b/6.19.6-custom-tux/linux
initrd     /b473cb59c64b4556949412573488670b/6.19.6-custom-tux/initrd
```
<br>
<img alt="Screenshot 2026-03-08 at 5 56 09 PM copy 2-2" src="https://github.com/user-attachments/assets/ea271698-558a-4fbe-9f6b-1fddac7f2bab" />



```bash
uname -a
Linux tux 6.19.6-custom-tux #1 SMP PREEMPT_DYNAMIC Sun Mar  8 15:36:41 CET 2026 x86_64 GNU/Linux
```

Optionally you can set the new kernel as the default option by adding the default like i did. 

```bash
cat /boot/loader/loader.conf
default b473cb59c64b4556949412573488670b-6.19.6-custom-tux.conf
timeout 3
#console-mode keep
```

## Trouble shooting

If you dont see Tux at boot make sure you have removed **quiet** and **loglevel=3** from your kernel cmd line (and you have added **fbcon=nodefer**). If this does not work make to remove kms from HOOKS in **/etc/mkinitcpio.conf** and run **sudo mkinitcpio -P** after.


## Contributing

Hi there! i would love if someone could add instructions on how to add the boot entries for other bootloaders then systemd boot (be cause systemd boot automates the process other bootloaders might need a more manual aproach).

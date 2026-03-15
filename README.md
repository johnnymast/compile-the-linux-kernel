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

<p>If you look at the last part of the kernel version string, you can see that it ends with arch1‑1. For this experiment, I want my custom kernel to use a different local version suffix — specifically -custom-tux. This helps distinguish it from the stock Arch kernel once it is installed.</p>

<p>To modify the kernel name and enable the Tux boot logo, open the kernel configuration menu:</p>


```bash
nake menuconfig
```


In the build menu, navigate to **General setup** → **Local version** and enter the custom suffix you want to use for your kernel. This value will be appended to the kernel version string, making it easier to distinguish your custom build from the stock Arch kernel.

<img alt="20260308_14h39m35s_grim" src="https://github.com/user-attachments/assets/44695eeb-4bd0-4627-b299-5b7ff6be320d" />

<br/>After setting the local version, use **Exit** until you return to the top‑level menu (you will know you are there when you see General setup listed at the top). From here, navigate to **Device Drivers** → **Graphics support** to enable the **Bootup logo** option.

<br>This section allows you to enable the classic Tux logo that appears during early boot. The menu should look similar to the screenshot below.

<br><img  alt="20260308_14h52m11s_grim" src="https://github.com/user-attachments/assets/d6c03918-6cb9-49d7-998d-fad9bd217a93" />


<br>Still within the **Graphics support** menu, go to **Frame buffer** Devices → Support for frame buffer device drivers and make sure the following options are enabled:


- Vesa VGA graphics support
- EFI-based Framebuffer support

<p>These options ensure that the kernel can display the Tux logo during early boot on systems using EFI firmware.</p>
<p>After enabling these settings, exit back to the main menu and make sure to save your configuration before leaving menuconfig.</p>

```bash
grep -E "LOGO|FB_SIMPLE|FB_EFI" .config

CONFIG_SYSFB_SIMPLEFB=y
CONFIG_FB_EFI=y
CONFIG_LOGO=y
CONFIG_LOGO_LINUX_MONO=y
CONFIG_LOGO_LINUX_VGA16=y
CONFIG_LOGO_LINUX_CLUT224=y
```
<p>If your output matches these entries, the logo and framebuffer support are correctly configured.</p>

<p>With the configuration validated, it is time to start building the kernel. Depending on your hardware, this process may take a significant amount of time; on my system it takes quite a while.</p>

??
To measure how long the kernel compilation takes, you can wrap the build command with ***time***:
??

```bash
time make -j$(nproc)
```


> ℹ️ **Info**
> On my system, compiling the Linux kernel took 20 minutes and 40 seconds.


If you do not want to measure the compile time, simply run:

```bash
make -j$(nproc)
```

<p>Compiling the kernel can take a while depending on your hardware. It’s a good moment to grab a coffee or watch something while the build completes. On most systems, expect anywhere between ***20–30*** minutes.</p>

<p>Once the build finishes, install the kernel modules and the kernel itself:</p>

```
sudo make modules_install
sudo make install
```
These commands will place the modules under ***/usr/lib/modules/*** and install the kernel, System.map, and initramfs into ***/boot***.
<br>

## Preparing the systemd‑boot entry for your custom kernel

After running make install, **systemd‑boot** automatically generates a new boot entry for the freshly compiled kernel. To ensure that the Tux logo is actually visible during boot, a few adjustments to this entry are required.
<br>
You can find the generated entry under **/boot/loader/entries**:

```bash
ls /boot/loader/entries                  
 arch.conf   b473cb59c64b4556949412573488670b-6.19.6-custom-tux.conf
```

The second file is the entry created for the custom kernel. There are a few important details to note when editing this file:

- I added mitigations=off to improve performance.
This disables CPU vulnerability mitigations, which reduces security, so use it only if you understand the risks.
- I removed quiet and loglevel=3.
These options suppress boot messages, which also suppresses the Tux logo.
- I added fbcon=nodefer.
Without this option, the framebuffer console initializes too late, causing the Tux logo not to appear during early boot.

These adjustments ensure that the framebuffer is active early enough and that the kernel does not hide the boot logo.


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

<br>

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

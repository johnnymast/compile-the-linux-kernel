[![Badge](https://img.shields.io/badge/Powered--by-NeuroSama-brightgreen?style=for-the-badge)](https://www.genome.gov/)


## Temporarly markdown markup (lol)

> ℹ️ **Info**
> Bleep
> 


# Compile the Linux kernel


[<img src="https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png" align="right" width="250">](https://www.archlinux.org/)


This is a personal experiment of mine to compile and install the linux kernel with the tux logo at boot. This project was done on arch linux on march 8th 2026. This story is much a work in progress and will be properly edited and fixed at a later date.
Below you can see the configuration of my machine as of before i started this project. Oh an important thing to note is that i am using systemd boot as a bootloader. Once you are done building your kernel you could use research of your own if you dont use systemd boot your self.

<br />
<br />
<br />



<img width="1444" height="1024" alt="20260308_13h33m57s_grim" src="https://github.com/user-attachments/assets/06ec15ef-31aa-4f7c-a8b9-0255c70ef44f" />

<br /><br />



Before we start wee first need to install some requirted dependencies so we can compile the kernel successfully.

```bash
sudo pacman -Syu
sudo pacman -S base-devel git ncurses flex bison openssl elfutils bc cpio
```

With that done the next step will be to get the Linux source code and extract it somewhere. 

```bash
mkdir -p ~/kernel && cd ~/kernel
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.19.6.tar.xz
tar xf linux-6.19.6.tar.xz
cd linux-6.19.6
```

And copy the config file from the current kernel into the new kernel's build path.

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

<img width="1332" height="781" alt="20260308_14h39m35s_grim" src="https://github.com/user-attachments/assets/44695eeb-4bd0-4627-b299-5b7ff6be320d" />

<br>Now use Exit to navigate back to the root of the menu (you will know you reached it if you see General Setup on top). Next navigate to **Device Drivers** &rarr; **Graphics support** to enable the **Bootup logo** option. You will see a page looking like this.
<br><img width="1333" height="1093" alt="20260308_14h52m11s_grim" src="https://github.com/user-attachments/assets/d6c03918-6cb9-49d7-998d-fad9bd217a93" />
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


```
time make -j$(nproc)
```


> ℹ️ **Info**
> Compiling the Linux kernel on my machine took 20 minutes and 40 seconds. 
> 


of if you dont want to messure the compile time use

```bash
make -j$(nproc)
```

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

This is the boot entry the i have of my custom kernel. There are a few things to not here one i haved added **xx** to make my pc faster (this will disable cpu exploit protections so we warned) and i made sure that i removed **quiet** and **loglevel=3** from my 
 
... TO BE CONTINUED ...


Optionally you can set the new kernel as the default option by adding the default like i did. 

```bash
cat /boot/loader/loader.conf
default b473cb59c64b4556949412573488670b-6.19.6-custom-tux 
timeout 3
#console-mode keep
```

``bash
default b473cb59c64b4556949412573488670b-6.19.6-custom-tux 
timeout 3
#console-mode keep
```

## Trouble shooting

If you dont see Tux at boot make sure you have removed **quiet** and **loglevel=3** from your kernel cmd line (aka the kernel parameters). If this does not work make to remove kms from HOOKS in **/etc/mkinitcpio.conf** and run **sudo mkinitcpio -P** after.


## Contributing

Hi there! i would love if someone could add instructions on how to add the boot entries for other bootloaders then systemd boot (be cause systemd boot automates the process other bootloaders might need a more manual aproach).

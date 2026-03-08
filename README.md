[![badge](https://img.shields.io/badge/Powered--by-NeuroSama-brightgreen?style=for-the-badge)](https://google.com](https://www.youtube.com/@Neurosama)

# Compile the Linux kernel
This is a personal experiment of mine to compile and install the linux kernel with the tux logo at boot. This project was done on arch linux on march 8th 2026.

> ℹ️ **Info**
> Bleep
> Bleep again




<img width="1444" height="1024" alt="20260308_13h33m57s_grim" src="https://github.com/user-attachments/assets/06ec15ef-31aa-4f7c-a8b9-0255c70ef44f" />

Alright before we start wee first need to install the dependencies we are going to need to compile the kernel successfully.

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
🐧 ❯ cp /usr/lib/modules/$(uname -r)/build/.config .config

~/kernel/linux-6.19.6 
🐧 ❯ make oldconfig

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

~/kernel/linux-6.19.6 took 2s 
🐧 ❯
```

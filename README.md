# compile-the-linux-kernel
This is a personal experiment of mine to compile and install the linux kernel with the tux logo at boot. This project was done on arch linux on march 8th 2026.

<img width="1444" height="1024" alt="20260308_13h33m57s_grim" src="https://github.com/user-attachments/assets/06ec15ef-31aa-4f7c-a8b9-0255c70ef44f" />



## Dependencies

```bash
sudo pacman -Syu
sudo pacman -S base-devel git ncurses flex bison openssl elfutils bc cpio
```


## Fetch the linux kernel code

```bash
mkdir -p ~/kernel && cd ~/kernel
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.19.6.tar.xz
tar xf linux-6.19.6.tar.xz
cd linux-6.19.6
```

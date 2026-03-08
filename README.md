# compile-the-linux-kernel
This is a personal experiment of mine to compile and install the linux kernel with the tux logo at boot.

## Dependencies

```bash
sudo pacman -Syu
sudo pacman -S base-devel git ncurses flex bison openssl elfutils bc cpio
```


## Fetch the linux kernel code

```bash
mkdir -p ~/kernel && cd ~/kernel
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.12.tar.xz
tar xf linux-6.12.tar.xz
cd linux-6.12
```

# TL-WN725N_V2 -> Odroid XU4
>Note : Tested on Odroid XU4 - Kernel `4.14.5-92`
`0bda:8179 Realtek Semiconductor Corp. RTL8188EUS 802.11n Wireless Network Adapter`
Odroid XU4 image can be download from here [Odroid Ubuntu Image](https://odroid.in/ubuntu_16.04lts/)

Personally i used this latest image [Ubuntu 16.04.3-4.14](https://odroid.in/ubuntu_16.04lts/ubuntu-16.04.3-4.14-mate-odroid-xu4-20171212.img.xz)

This tutorial based on [this forum topic](https://forum.odroid.com/viewtopic.php?f=52&t=1674)
and modified by Giovanni Bruno [Repo](https://github.com/gbr1/erwhi/wiki/Setup-TP-link-722n-v2-on-Odroid-XU4)

>NOTE: do everything as superuser (sudo su)


### 1. Kernel Updates (Only do Kernel updates while your Kernel system not `4.14.+`) [Kernel Building](https://wiki.odroid.com/odroid-xu4/software/building_kernel)
```
sudo apt install git gcc g++ build-essential
git clone --depth 1 https://github.com/hardkernel/linux -b odroidxu4-4.14.y
cd linux
make odroidxu4_defconfig
make -j8
sudo make modules_install
sudo cp -f arch/arm/boot/zImage /media/boot
sudo cp -f arch/arm/boot/dts/exynos5422-odroidxu3.dtb /media/boot
sudo cp -f arch/arm/boot/dts/exynos5422-odroidxu4.dtb /media/boot
sudo cp -f arch/arm/boot/dts/exynos5422-odroidxu3-lite.dtb /media/boot
sudo sync
sudo reboot
```


### 2. Kernel Sources in /usr/src/
```
wget https://github.com/hardkernel/linux/archive/odroidxu4-4.14.y.zip
mv odroidxu4-4.14.y.zip /usr/src/linux.zip
cd /usr/src
7z x -y linux.zip
ln -s linux-odroidxu4-4.14.y linux
```

### 3. check kernel version
NOTE: extraversion should be with minus e.g. `4.14.5-92` -> extraversion is `-92`
```
cd /usr/src/linux
uname -a
```
check Makefile version
```
head Makefile
```
add Makefile extraversion = `-92`
```
nano Makefile
```

### 4. Copy Required Configuration
```
ll arch/arm/configs
cp arch/arm/configs/odroidxu4_defconfig ./.config
cp /usr/src/linux-headers-$(uname -r)/Module.symvers /usr/src/linux
```

### 5. Install Ncurses Library
apt-get install libncurses5-dev libssl-dev


### 6. Run Kernel Source Configuration
>NOTE: just exit
```
make menuconfig
make prepare
make modules_prepare
```

### 7. Download Driver
Source from Github [lwfinger](https://github.com/lwfinger/rtl8188eu)
```
git clone https://github.com/ezracb/TL-WN725N_V2.git
```

### 8. Build
> Odroid XU4 has limited resources of Memory so here is the way to compile the driver
```
cd rtl8188eu
make clean
CONFIG_RTL8188EU=m make -C /usr/src/linux M=`pwd`
```

### 9. Copy the build information into the kernel drivers
```
cp 8188eu.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless/
depmod -a
modprobe 8188eu
```
>NOTE: if errors appear use dmesg


### 10. Test Wifi
```
lsmod
iwlist wlan0 scan
```

### 11. Configure
```
nmtui
check using
nmcli c
ifconfig -a
```

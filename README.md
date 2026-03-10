## Bài tập HDH Nhúng: UBoot và Kernel
# I. Chuẩn bị môi trường
 1. Phần cứng

- BeagleBone Black (BBB)

- Thẻ microSD ≥ 4GB

- USB-TTL UART (CP2102 / FT232 / CH340)

2. Dây nối:

- GND

- RX

- TX
# II. Tải Toolchain
```
wget https://developer.arm.com/-/media/Files/downloads/gnu-a/10.3-2021.07/binrel/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz

tar -xf gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf.tar.xz
```
Thêm vào PATH
```
export PATH=$PATH:$PWD/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf/bin
export CROSS_COMPILE=arm-none-linux-gnueabihf-
export ARCH=arm
```
# III. Biên Dịch U-Boot
Clone source:
```
git clone https://github.com/u-boot/u-boot.git
cd u-boot
```
Cấu hình cho BBB
```
make am335x_evm_defconfig
```
Build
```
make -j4
```
Sau khi build thành công sẽ có :
```
MLO
u-boot.img
```
👉 Đây chính là file cần ghi vào thẻ nhớ.
# IV. Tạo phân vùng thẻ nhớ
Cắm SD card vào Ubuntu.

Xác định thiết bị:
```
lsblk
```
Mở fdisk:
```
sudo fdisk /dev/sdb
```
Partition 1

- FAT32

- 100MB

- Bootable

Partition 2

- ext4

- phần còn lại

Format:
```
sudo mkfs.vfat /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
```
# V. Copy U-Boot vào thẻ
Mount partition:
```
sudo mount /dev/sdb1 /mnt
```
Copy file:
```
cp MLO /mnt/
cp u-boot.img /mnt/
sync
```
Unmount
```
sudo umount /mnt
```
# VI. Kết nối UART Debug
Mở terminal:
```
sudo minicom -D /dev/ttyUSB0 -b 115200
```
# VII. Boot U-Boot
Cắm SD vào BBB và cấp nguồn.
Terminal sẽ hiện version của nó :
```
U-Boot 2023.04 (Feb 26 2026 - 00:58:28 +0700)

arm-linux-gnueabi-gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
GNU ld (GNU Binutils for Ubuntu) 2.34
```
👉 Đạt yêu cầu phần U-Boot khởi động thành công

# VIII. Biên dịch Linux Kernel

Clone kernel:
```
git clone https://github.com/beagleboard/linux.git
cd linux
```
Config cho BBB:
```
make bb.org_defconfig
```
Build:
```
make -j4 zImage
make -j4 dtbs
```
File thu được:
```
arch/arm/boot/zImage
arch/arm/boot/dts/am335x-boneblack.dtb
```
# IX. Copy Kernel vào thẻ

Mount SD:
```
sudo mount /dev/sdb1 /mnt
```
Copy:
```
cp arch/arm/boot/zImage /mnt/
cp arch/arm/boot/dts/am335x-boneblack.dtb /mnt/
sync
```
Unmount
```
sudo umount /mnt
```
# X. Boot Kernel từ U-Boot

Vào terminal U-Boot.

Load kernel:
```
load mmc 0:1 0x82000000 zImage
```
Load device tree:
```
load mmc 0:1 0x88000000 am335x-boneblack.dtb
```
Boot:
```
bootz 0x82000000 - 0x88000000
```
# XI. Kết quả mong đợi

Terminal hiển thị:
```
Starting kernel ...

Linux version ...
CPU: ARMv7
Machine model: BeagleBone Black
```
Sau đó:
```
Waiting for root filesystem
```
👉 Đây chính là yêu cầu:
✔ Hiển thị thông tin Kernel
✔ Chờ rootfs





# Mainline U-Boot on MediaTek mt6582

Works uart, sd-card.

## Build
```
make mt6582_prestigio_pmt5008_3g_defconfig
make -jx
```
## Installing in "secondary" bootloader mode
```
# Create a dummy initrd so that the stock bootloader will accept it
dd if=/dev/random of=ramdisk bs=2048 count=9

# Append a MediaTek header so UBoot will pass a check
mkimage ramdisk ROOTFS > ramdisk.img.mtk

mkimage u-boot.bin KERNEL > u-boot.bin.mtk

# You may need to select other parameters 
mkbootimg-osm0sis --base 10000000 --pagesize 2048 --kernel_offset 00008000 --ramdisk_offset 01000000 --tags_offset 00000100 --second_offset  00f00000  --kernel u-boot.bin.mtk --ramdisk ramdisk.img.mtk  -o u-boot-mt6582.img
```
Using the SP Flash Tool, flash to the "BOOTING" section.

## Installing in "first" bootloader mode

Find the u-boot-mtk.bin file and flash it with the SP Flash Tool to the "UBOOT" section.

ATTENTION: In this mode, the screen will not work, you can see the work of u-boot only by uart.

# Usage

## Running test builds of u-boot.bin by uart
Install ckermit and write a script
```
cat uartboot-mt6582-uboot.script
#!/usr/bin/ckermit

set port /dev/ttyUSB0
set speed 921600
set carrier-watch off
set flow-control none
set prefixing all

echo {loading u-boot}
PAUSE 1

OUTPUT loadb 0x81e00000 921600\{13}
send /home/username/build/../u-boot/u-boot.bin
PAUSE 1
OUTPUT go 0x81e00000\{13}
c
```
## Download and run linux kernel by uart
Write a script for ckermit
```
cat uartboot-mt6582-linux.script
#!/usr/bin/ckermit

set port /dev/ttyUSB0
set speed 921600
set carrier-watch off
set flow-control none
set prefixing all

OUTPUT setenv initrd_high 0x85000000\{13}

echo {loading zImage}
PAUSE 1

OUTPUT loadb ${loadaddr} 921600\{13}
send /home/username/build/linux/../zImage

echo {loading fdt}
PAUSE 1

OUTPUT loadb ${fdt_addr_r} 921600\{13}
send /home/username/build/linux/../mt6582-device.dtb

echo {loading ramdisk}
PAUSE 1

OUTPUT loadb 0x85000000 921600\{13}
send uInitrd
PAUSE 1
OUTPUT bootz ${loadaddr} 0x85000000 ${fdt_addr_r}\{13}
c
```

## Booting linux kernel from sdcard
Write **boot.cmd** as below
```
env set initrd_high 0x85000000
load mmc 1:1 0xac000000 mt6582-prestigio-pmt5008-3g.dtb
load mmc 1:1 0x84000000 zImage
load mmc 1:1 0x85000000 uInitrd
bootz 0x84000000 0x85000000 0xac000000
```

Sign it with mkimage for U-Boot
`mkimage -C none -A arm -T script -d boot.cmd boot.scr`
And put the **boot.scr** file in the root of the sdcard.

Don't forget to include other files too!

Note: uInitrd is created by the command `mkimage -A arm -T ramdisk -C none -n uInitrd -d your_ramdisk  uInitrd`


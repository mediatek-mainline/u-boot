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


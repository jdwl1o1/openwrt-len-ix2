# openwrt for lenovo /iomega ix2-dl
openwrt for the Lenovo / Iomega IX2-DL NAS

This project includes the files needed to compile OpenWRT for the lenovo / Iomega ix2-dl NAS

details of openwrt are available at openwrt.org  

Files and images are provided 'as is' with no warranty or guarantee.

License is GPL2, unless otherwise specified in the files. No ownership is claimed over work of others.

Thanks to Bodhi, Luo and hippi-viking @ forum.doozan.com for dts & u-boot assistance.

Lenovo ix2-dl is a dual SATA NAS powered by a Marvell
 Kirkwood SoC clocked at 1.6GHz. It has 256MB of RAM and 1GiB of
 flash memory, 1x USB 2.0 and 1x 1Gbit/s NIC. 
 Was also sold by Iomega as the Iomega StorCentre ix2-dl.
 This device is very similar to the ix2-200 however it lacks 2 USB ports and includes 
 a larger nand flash unit. The IX2-DL was sold without drives, with the OS stored in nand.
```
Specification:
- SoC: Marvell Kirkwood 88F6282
- CPU/Speed: 1600Mhz
- Flash-Chip: Samsung NAND
- Flash size: nand: 1024 MiB, SLC, erase size: 128 KiB, page size: 2048, OOB size: 64
- RAM: 256MB
- LAN: 1x 1000 Mbps Ethernet
- WiFi: none
- 1x USB 2.0
- UART1: for serial console. UART0 not available.
```
Installation instructions (modified from the ix2-200): 
This runs OpenWrt from ram to allow sysupgrade to install to nand.

1. download initramfs-uImage image and sysupgrade image, copy into tftp server or ext2 formatted usb drive
2. access uboot environment with serial cable and run
```
    setenv mainlineLinux yes
    setenv console 'console=ttyS0,115200'
    setenv mtdparts 'mtdparts=orion_nand:0x100000@0x000000(u-boot)ro,0x20000@0xA0000(u-boot environment)ro,0x300000@0x100000(kernel),0x1C00000@0x400000(ubi)'
    setenv owrtboot 'nand read.e 0x800000 0x100000 0x300000;; setenv bootargs $(console) $(owrt_bootargs_root); bootm 0x800000'
    setenv owrt_bootargs_root 'root='
    setenv bootcmd 'run owrtboot'
    saveenv
```
For USB Boot:   
```
 usb reset; ext2load usb 0:1 0x00800000 /[initramfs image]; bootm 0x00800000
```
For TFTP boot:
```
    setenv serverip [tftp server ip]    
    setenv ipaddr 192.168.1.13
    tftpboot 0x00800000 [initramfs image]
    bootm 0x00800000
```
3. ssh to openwrt and sysupgrade to install into flash
```
    mkdir /mnt/usb
    mount /dev/sda1 /mnt/usb
    cp /mnt/usb/*sysupgrade* /tmp   
    sysupgrade -n /tmp/sysupgrade.bin
```
4. access openwrt by dhcp ip address assigned by your router or at 192.168.1.1

notes;
1 - sata drives should not be installed when installing & any data will not be accessible until LVM and mdadm modules are installed.

2 - this is a 1 way process. Once the nand is overwritten returning to stock will not be possible.

3 - to access u-boot variables from within openwrt add the following lines to /etc/fw_env.config in openwrt
```
/dev/mtd1 0x0000 0x20000 0x20000
/dev/mtd2 0x0000 0x20000 0x20000
```

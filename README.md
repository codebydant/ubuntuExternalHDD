# Ubuntu to External HDD
This is a tutorial to install Ubuntu in an external HDD as a portable/bootable-usb.

## PC Requirements
This tutorial is for an UEFI laptop without secureBoot.

-	Legacy mode: OFF
-	Secure boot: OFF
-	UEFI: ON
-	USB-boot: ON

## Software
For this tutorial we need to create an USB-bootable device with the target Ubuntu image (Ubuntu 20.04 in this case).   

- Rufus: https://rufus.ie/
- Ubuntu iso image: https://ubuntu.com/download/desktop

## Tutorial
The procedure for this tutorial is perfomed by 3 steps. 

1.	USB-bootable device
2.	Configure external HDD
3.	Install Ubuntu into external HDD
4.	Install/Configure grub-bootloader into external HDD
5.	Remove grub-bootloader in windows host PC

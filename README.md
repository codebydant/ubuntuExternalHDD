# Ubuntu to External HDD
This is a tutorial to install Ubuntu in an external HDD as a portable/bootable-usb. It is based in the post "HOW TO CREATE A TRULY PORTABLE UBUNTU INSTALLATION ON AN EXTERNAL USB HDD OR SSD" at [58Bits article](https://www.58bits.com/blog/2020/02/28/how-create-truly-portable-ubuntu-installation-external-usb-hdd-or-ssd).

## PC Requirements
This tutorial is for an UEFI laptop without secureBoot.

-	Legacy mode: OFF
-	Secure boot: OFF
-	UEFI: ON
-	USB-boot: ON

## What you will need
For this tutorial we need 2 things: 

1.	**A bootable USB thumb drive with the Ubuntu target installation media**<br/>
	You can create this from a Windows computer using [Rufus](https://rufus.ie/) and with the [Ubuntu iso image](https://ubuntu.com/download/desktop). See [How to Create a Bootable Linux USB Flash Drive](https://www.youtube.com/results?search_query=linux+bootable+usb). Check that you can boot from this thumb drive on the computer you plan on working from. Choose the *‘Try Ubuntu’* option.

2.	**Your target portable external hard drive**<br/>
	Note that we’ll re-partition this drive, so backup any data that may already be on the drive as it will be lost during re-partitioning.

## Getting Started
1.	Boot your computer using the thumb drive prepared above, and choose ‘Try  Ubuntu’.
2.	Plug in your target portable external hard drive.
3.	Start GParted. GParted is the partition manager application we’re going to use to prepare the portable hard drive. Once GParted is started - in the upper right corner, change the target drive to your external portable drive. It’s important that you correctly identify this drive as we’re going to re-partition the drive. In the screenshot below - my external and portable drive is identified as /dev/sdb - and it is currently unpartitioned. Unmount (right click and unmount) any currently mounted partitions on this drive and delete all partitions (again - be double sure you’re working on the correct drive).

![Start GParted](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/gparted-01.png?raw=true)

## Preparing the portable drive
We’re going to create three new partitions on the target external drive.

1.	Again using GParted, right click on the unallocated volume, choose New, and create a 100MB fat32 partition. Click on the green checkmark to apply the pending operation. Once the partition has been created - right click on the newly created partition and select ‘Manage Flags’. Enable the boot and esp flags. When we're done, this partition will become the system ‘boot’ partition, and will include EFI information including the GNU GRUB boot loader. In fact, creating this partition as a working boot volume under EFI using GRUB is the heart of our problem in trying to create a truly portable external OS drive, and so there are a few more steps to complete before we can achieve this.

2.	Next create an 8GB linux-swap partition. The size of your swap partition may vary and so you’ll need to do a little research to determine the appropriate size for your expected workload. A rule of thumb for modern personal computers with plenty of RAM is to create a swap partition about ½ the size of available RAM if you DO NOT plan on supporting full hibernate (most computers will still suspend or sleep fine).

3.	Finally, create the main or root / partition for our target portable drive. Create an ext4 partition of whatever size you require for your system. Apply all pending operations and you should now have a disk partition layout that looks similar to the screenshot below. [Note that in my case I still have about 500GB unallocated space as this is a 1TB external drive. Also note that in my ‘real’ setup I created a 64GB ext4 volume for Ubuntu OS, and then when everything was up and running, I created an additional 256GB ext4 volume which I then encrypted with LUKS, and mounted as my /home directory]

![My final partition arrangement on the external hard drive](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/gparted-02.png?raw=true)

With the external drive all prepared, we now need to make a couple of notes, specifically -  note the device and partition numbers. In this example my external drive is identified as /dev/sdb with partitions located on /dev/sdb1 (fat32 system/boot), /dev/sdb2 (linux-swap), /dev/sdb3 (ext4 root volume). We also need to record the UUIDs of the system and root volumes for this drive. Double click on the fat32 system partition at /dev/sdb1, and from the ‘Information about’ screen that pops up - make a note of the UUID. In my case: ED3C-7CB8. Now do the same for the root volume - the ext4 partition on /dev/sdb3 - double click on the partition and note the UUID. In my case: dd8eed75-c315-420f-b208-92301cfbf300.

We’re now almost ready to install Ubuntu 19.10 on this drive. Note first however, that in two attempts at this process, the system volume of the computer I was using for this process (my Windows 10 computer) was modified and left with a dual boot installation, which is NOT what we want (as that would effectively ‘bind’ our external hard drive to this computer). When we install Ubuntu 19.10 - we’ll mainly follow the instructions here - How to install Ubuntu on portable external Hard Drive? - however, during installation - Ubuntu 19.10 will use the first UEFI system partition it finds to install the modified bootloader, and so the instructions in the previous link that specify the following: “Very important: change the installation of the bootloader to the USB HD. This will most likely be /dev/sdb. This will prevent you from overwriting the master boot record on your hard drive. (If you do this by accident, it's easily fixed).” - simply won’t work. The only scenario I’ve not yet tried to prevent this is unplugging, or removing the computer’s internal hard drive before installing Ubuntu onto our target external drive. The remaining steps will show how to fix this, as well as how to correctly install a working GRUB bootloader onto our newly created /dev/sdb1 system fat32 ESP partition.

## Install ubuntu into external HDD

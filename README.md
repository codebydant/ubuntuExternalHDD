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
3.	Start GParted. GParted is the partition manager application we’re going to use to prepare the portable hard drive. Once GParted is started - in the upper right corner, change the target drive to your external portable drive. It’s important that you correctly identify this drive as we’re going to re-partition the drive. In the screenshot below - my external and portable drive is identified as */dev/sdb* - and it is currently unpartitioned. Unmount *(right click and unmount)* any currently mounted partitions on this drive and delete all partitions (again - be double sure you’re working on the correct drive).

![Start GParted](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/gparted-01.png?raw=true)

## Preparing the portable drive
We’re going to create three new partitions on the target external drive.

1.	Again using GParted, right click on the unallocated volume, choose New, and create a **100MB fat32** partition. Click on the green checkmark to apply the pending operation. Once the partition has been created - right click on the newly created partition and select ‘Manage Flags’. Enable the **boot** and **esp** flags. When we're done, this partition will become the system *‘boot’* partition, and will include EFI information including the GNU GRUB boot loader. In fact, creating this partition as a working boot volume under EFI using GRUB is the heart of our problem in trying to create a truly portable external OS drive, and so there are a few more steps to complete before we can achieve this.

2.	Next create an XGB linux-swap partition, see [Linux DiskSpace](https://help.ubuntu.com/community/DiskSpace) for more information. The size of your swap partition may vary and so you’ll need to do a little research to determine the appropriate size for your expected workload. A rule of thumb for modern personal computers with plenty of RAM is to create a `swap partition = 2*the size of available RAM` if you do plan on supporting full hibernate (most computers will still suspend or sleep fine). For this tutorial, I have a 8GB RAM, so I chose `linux-swap partition=16GB+500MB` to be sure.

3.	Finally, create the main or root `/` partition for our target portable drive. Create an ext4 partition of whatever size you require for your system. Apply all pending operations and you should now have a disk partition layout that looks similar to the screenshot below. [Note that in my case I still have about 500GB unallocated space as this is a 1TB external drive. 

![My final partition arrangement on the external hard drive](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/gparted-02.png?raw=true)

With the external drive all prepared, we now need to make a couple of notes, specifically -  note the device and partition numbers. In this example my external drive is identified as /dev/sdb with partitions located on /dev/sdb1 (fat32 system/boot), /dev/sdb2 (linux-swap), /dev/sdb3 (ext4 root volume). We also need to record the UUIDs of the system and root volumes for this drive. Double click on the fat32 system partition at /dev/sdb1, and from the ‘Information about’ screen that pops up - make a note of the UUID. In my case: ED3C-7CB8. Now do the same for the root volume - the ext4 partition on /dev/sdb3 - double click on the partition and note the UUID. In my case: dd8eed75-c315-420f-b208-92301cfbf300.

We’re now almost ready to install Ubuntu on this drive. Note first however, that in two attempts at this process, the system volume of the computer I was using for this process (my Windows 10 computer) was modified and left with a dual boot installation, which is NOT what we want (as that would effectively ‘bind’ our external hard drive to this computer). When we install Ubuntu on an external HDD is very important to change the installation of the bootloader to the USB HD. This will most likely be /dev/sdb. This will prevent you from overwriting the master boot record on your hard drive. The remaining steps will show how to correctly install a working GRUB bootloader onto our newly created /dev/sdb1 system fat32 ESP partition.

## Configure external HDD
With your external target drive all prepared we’re now ready to install Ubuntu 19.10. As per the link in the previous section, we’re going to start a normal installation followed by ‘Something else’ when we get to the partition selection step. You should still be booted from your Ubuntu Installation media thumb drive.

Close GParted and then double click on the Install Ubuntu 19.10 icon on your desktop. Choose your language settings, and optionally install third-party drivers. On the next screen, for ‘Installation Type’ - choose the last option ‘Something else’ before proceeding.

![3](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/ubuntu-installation-01.png?raw=true)

Now that we’re on the ‘Something else’ installation type screen - scroll down the list of available drive volumes until you see your device and the partitions we previously created. In this example /dev/sdb1, /dev/sdb2, and /dev/sdb3.

1.	Double click on the 100MB fat32 system efi partition we created (/dev/sdb1)and choose ‘Use as EFI system partition’ but do not format the partition.

2.	Double click on the /dev/sdb2 partition and choose ‘Use as swap area’.

3.	Then double click on the /dev/sdb3 partition - and choose use as 'Ext4 journaling file system’, and set the mount point to / or root, and again do not format this partition.

4.	Lastly - select the ‘Device for boot loader installations:’ to the name of the device for your external hard drive (although as noted above, this may not work and you’ll need to follow the remaining steps below). 
Your settings should look like the following:

![4](https://github.com/danielTobon43/ubuntuExternalHDD/blob/master/examples/ubuntu-installation-02.png?raw=true)

Go ahead now and install Ubuntu 19.10 to your external drive - setting your timezone, and user account information as you would with a normal Ubuntu installation.

## Install ubuntu into external HDD
Go ahead now and install Ubuntu 19.10 to your external drive - setting your timezone, and user account information as you would with a normal Ubuntu installation.


## Install grub onto the ESP partition
As previously mentioned, the Ubuntu 19.10 installation process will likely have created a ‘dual boot’ installation by modifying your host computer's main EFI / ESP partition, effectively ‘binding’ your external drive to this computer. If so, there are two remaining tasks.

First, we need to correctly install the Grub bootloader onto the boot partition of our external portable drive - turning it into a truly portable installation.

The second and last step will be to remove the ‘dual boot’ configuration from the computer you are using to create this new external and portable drive. You can check to see if any of this applies to you by rebooting your computer WITH the new external drive plugged in - but selecting your computers primary disk (not the external disk) to boot from. If you see a ‘dual boot’ Grub option screen - then everything that follows applies.

Make sure you’re now booted from the Ubuntu installation thumb drive (in the ‘Try Ubuntu mode), and start the ‘Terminal’ application.

First we’re going to unmount the media volume of the thumb drive (leaving ‘Try Now’ Ubuntu running in memory only). Replace the 'uuid of your media' text below with the uuid in your system. There should be only one under media/ubuntu.

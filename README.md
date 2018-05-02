## Table of contents

[1. Create the bootable usb](#Create-the-Bootable-usb)



[2. Connect to a wifi](/NETWORK.md)

[3. Partition the disk](/PARTITION.md)

[4. Mount a file system](FILE-SYSTEM.md)

[5. Arch Installation](INSTALL.md)

[6. Configure the system](CONFIGURE.md)

[7. Boot loader](BOOT-LOADER.md)

<a name="abcd"></a> where you want to link to and refer to it on the same page by [link text](#abcd)


## Summary

This week my goal was to install Arch on my old 2012 macbook air. I decide to make a guide to help me really understand what i was doing.

# Create the Bootable usb 
The first step was obviously to create a bootables usb key.

1. Download the ISO of Arch. Multiple download link, depending on your country can be found [here](https://www.archlinux.org/download/)

2. Since i'm using a mac i was able to copy the ISO using dd. I've follow the instruction on the [wiki](https://wiki.archlinux.org/index.php/USB_flash_installation_media), Here's a Summary

    We first need to see wich USB device is connected to our computer. It's possible to do so by using

    ```diskutil list```

    After knowing wich device we would like to copy the ISO to it's important to unmount this device

    ```diskutil unmountDisk /dev/diskX```

    You muste replace the X with the proper device number. Im my case it was 2.

    Finally you can use dd to copy the ISO file

    ```dd if=/to/arch.iso of=/dev/rdiskX bs=1m```

    >The dd utility shall copy the specified input file to the specified output file with possible conversions using specific input and output block sizes.<sup>[1](#ft1)</sup>

3. Once the USB is ready, reboot your mac on the USB.


#### Networking
The first thing you see when you boot is a console, and the first thing that came to my mind was that i need to be connected to internet. So in order to do so i've follow those step from the [wiki](https://wiki.archlinux.org/index.php/Wireless_network_configuration). Here's the summary

1. We must check if we have a drive for our network card. It is possible to list all PCI buses and device with

    ```lspci```

    If your network card is available you should see it in the list.

2. Once you know you have a network card it's time to check for the network interface.

    ```ip link```

    > ip link provides the ability to display link layer information, activate an interface, deactivate an interface, change link layer state flags, change MTU, the name of the interface, and even the hardware and Ethernet broadcast address.<sup>[2](#ft2)</sup>

    Your interface should start with a 'w', as a exemple, mine was wlp2s0b1

3. With your interface name it's time to set it up

    ```ip link set wlp2s0b1 up```

4. Insure that your interface is up

    ```ip link show wlp2s0b1```

    you should see something like that < XXX, XXX, UP > make sure the UP is there if not, that mean your interface isn't up

5. Now with our interface up it's possible to scan access point

    ```iw dev wlp2s0b1 scan | less```
    >iw is a new nl80211 based CLI configuration utility for wireless devices. It supports all new drivers that have been added to the kernel recently. The old tool iwconfig, which uses Wireless Extensions interface, is deprecated and it's strongly recommended to switch to iw and nl80211.<sup>[3](#ft3)</sup>

6. Next we are gonna use wpa_supplicant to connect to our Wifi

    ```wpa_supplicant -B -i wpl2s0b1 -c<(wpa_passphrase "MYSSID" "passphrase")```

    if that work correctly you should see a 'Successfully initialized wpa_supplicant'

    in this step we are creating a config with the command

    ```wpa_passphrase "MYSSID" "passphrase"```

    And pipe the config that this commande create to wpa_supplicant. It's also possible to create a config file with wpa_passphrase and passe this file instead of the command

    >wpa_supplicant is a WPA Supplicant for Linux, BSD, Mac OS X, and Windows with support for WPA and WPA2 (IEEE 802.11i / RSN). It is suitable for both desktop/laptop computers and embedded systems. Supplicant is the IEEE 802.1X/WPA component that is used in the client stations. I implements key negotiation with a WPA Authenticator and it controls the roaming and IEEE 802.11 authentication/association of the wlan driver.<sup>[4](#ft4)</sup>

7. The final step is to start the dhcp demaon that will assign use ip address

    ```dhcpcd wlp2s0b1```

    >dhcpcd runs on your machine and silently configures your computer to work on the attached networks without trouble and mostly without configuration.

8. Testing our brand new connection, let's ping google

    ```ping google.com```



#### Partition
You can do this before connecting to a network. This section doesn't require any internet.

1. First let's see our current partition

   ```lsblki -f```

   >List information about all available or the specified block devices.

   I fell like using fdisk give us better information if you want to list all the device.
   
   ```fdisk -l```

2. Next we are going to wipe the older partition table and create  a new one.
   
   ```gdisk /dev/sda```

   We need to be in expert mode

   ```> x ```

   ```> z ```
   
   We use the commande zap to: 'zap (destroy) GPT data structures and exit'
   After it's done we it enter for the next two question.

3. Let's create our partition

   ```cgdisk /dev/sda```

   >Upon start, cgdisk attempts to identify the partition type in use on the disk. If it finds valid GPT data, cgdisk will use it. If cgdisk finds a valid MBR or BSD disklabel but no GPT data, it will attempt to convert the MBR or disklabel into GPT form. (BSD disklabels are likely to have unusable first and/or final partitions because they overlap with the GPT data structures, though.) Upon exiting with the 'w' option, cgdisk replaces the MBR or disklabel with a GPT. This action is potentially dangerous! Your system may become unbootable, and partition type codes may become corrupted if the disk uses unrecognized type codes. Boot problems are particularly likely if you're multi-booting with any GPT-unaware OS. If you mistakenly launch cgdisk on an MBR disk, you can safely exit the program without making any changes by using the Quit option.<sup>[4](#ft4)</sup>


4. Presse any key to continue 

5. create a boot partition
 
   ```[ NEW ]```
   
   First sector ... 

   ```[ ENTER ]``` 

   Size Sector: The Arch linux installation recommende 550Mbi for the boot partition
   

   ```550Mib [ ENTER ]```

   Selection the type of partition. For the boot we want EFI System. It's possible to list all code by pressing L

   ```EF00```

   Enter new partition name:

   ```boot```

6. Create a swap partition. 

   > Swap space in Linux is used when the amount of physical memory (RAM) is full. If the system needs more memory resources and the RAM is full, inactive pages in memory are moved to the swap spac .<sup>[6](#ft6)</sup>
 
    
   ```[ NEW ]```
   
   First sector ... 

   ```[ ENTER ]``` 

   Size Sector: For the swape we are gonna use 8Gib.  
   

   ```8Gib [ ENTER ]```

   Selection the type of partition.For the swap the code is It's possible to list all code by pressing L

   ```8200```

   Enter new partition name:

   ```swap```

7. Creating root partition
   
   ```[ NEW ]```
   
   First sector ... 

   ```[ ENTER ]``` 

   Size Sector:  i've chose 15Gib.
   

   ```15Gib [ ENTER ]```

   Selection the type of partition.For the swap the code is It's possible to list all code by pressing L

   ```8300```

   Enter new partition name:

   ```root```

8. Creating our home partition

   
   ```[ NEW ]```
   
   First sector ... 

   ```[ ENTER ]``` 

   Size Sector:The remaining space 
   

   Selection the type of partition.For the swap the code is It's possible to list all code by pressing L

   ```8300```

   Enter new partition name:

   ```home```

9. Finaly to make our change we need to write it to the disk: 

   ```[ WRITE ]```

#### Mounting a file system


1. Unmount all mounted file system. 
 
   ```findmnt /dev/sda```

   > To list all mounted file systems (https://wiki.archlinux.org/index.php/File_systems#List_mounted_file_systems)

   ```unmount </dev/sdX>```

2. Making our file system to the boot partition. According to the Arch wiki, the boot partion must be mount with a FAT32 file system in order to work.

   ```# mkfs.vfat -F32 /dev/sd1```

3. Making the swape

   ```mkswap /dev/sda2```

   > Set up a Linux swap area on a device or in a file (man page)

   ```swapon /dev/sda2```

   > Swapon is used to specify devices on which paging and swapping are to take place (man page)
 

4. Making our file system for the root. 

   ```mkfs.ext4 /dev/sda3```

5. making our file system for the hoom

   ```mkfs.ext4 /dev/sda4```

6. Finaly mount our file system


   ```mount /dev/sda3 /mnt```


   ```mkdir /mnt/boot```


   ```mkdir /mnt/home```


   ```mount /dev/sda4 /mnt/boot``` 


   ```mount /dev/sda4 /mnt/home``` 

	   
#### Installation

In this section we are going to install the [base package](https://www.archlinux.org/groups/x86_64/base/)

1. Run the Arch installation script 

   ```pacstrap /mnt base```

#### Configure the system  

1. We need to generate the fstab

   ```genfstab -U /mnt >> /mnt/etc/sftab```

   >The fstab file can be used to define how disk partitions, various other block devices, or remote filesystems should be mounted into the filesystem.(https://wiki.archlinux.org/index.php/Fstab)
   
   ```nano /mnt/etc/fstab```
   
   > Make sure that the line of the ext4 partition ends with a “2”, the swap partition’s line ends with a “0”, and the boot partition’s line ends with a “1”. This configures the partition checking on boot.(https://medium.com/@philpl/arch-linux-running-on-my-macbook-2ea525ebefe3?token=Sa7zbfKtIQl2yvlJ)

2. We need to change root into the new system

   ```arch-chroot /mnt```

3. Setting the time zone

   * we need to maque a symlink 

      ```ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime```

      ```hwclock --systohc```

      > Administration tool for the hardware clock

4. Setting localization
   
   * We need to uncomment our locale in /etc/locale.gen

   * Generate the localization
      
      ```locale-gen```

5. Set the host name


   ```echo <hostname> > /etc/hostname```
   
   
   Add the entries to /etc/hosts
   
   
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	<hostname>.localdomain	<hostname>
```

6. Set the root password

   ```passwd```
   

#### Boot loader

A boot loader is the first program that runs when a computer start. It is responsible for selecting, loading and transferring control to an operatinf system kernel. After that the kernel initializes the rest of the operating system.(https://wiki.archlinux.org/index.php/GRUB)


We are using systemd-boot
systemd-boot, previously called gummiboot, is a simple UEFI boot manager which executes configured EFI images. The default entry is selected by a configured pattern (glob) or an on-screen menu. It is included with systemd, which is installed on Arch system by default.(https://wiki.archlinux.org/index.php/Systemd-boot)

1. Install the bootloader to our boot partition

   ```bootctl --path=/boot install```
   
   > The above command will copy the systemd-boot binary to /boot/EFI/BOOT/BOOTX64.EFI and add systemd-boot itself as the default EFI application (default boot entry) loaded by the EFI Boot Manager.(https://wiki.archlinux.org/index.php/mac#Setup_bootloader)
   
2. Install the Microcode for intel (intel-ucode)

   >Processor manufacturers release stability and security updates to the processor microcode (https://wiki.archlinux.org/index.php/Microcode)
   
   ```pacman -S  intel-ucode```
   
3. Create the Arch Linux boot entry

   ```nano /boot/loader/entries/arch.conf```
   
   Enter the configuration taking from the Arch linux (https://wiki.archlinux.org/index.php/Systemd-boot)
   
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=LABEL=arch_os rw
``` 

4. Configure systemd-boot to boot our config

   ```echo "default arch" > /boot/loader/loader.conf```
   
5. Finaly reboot our system

   ``èxit```
   
   ```reboot```
   
   

---
References

<a name="ft1">1 - dd</a>: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/dd.html

<a name="ft2">2 - ip</a>: http://linux-ip.net/html/tools-ip-link.html

<a name="ft3">3 - iw</a>: https://wireless.wiki.kernel.org/en/users/documentation/iw

<a name="ft4">4 - wpa_supplicant</a>: http://w1.fi/wpa_supplicant/

<a name="ft5">5 - cgdisk</a>: https://www.rodsbooks.com/gdisk/cgdisk.html

<a name="ft6">6 - cgdisk</a>: https://www.rodsbooks.com/gdisk/cgdisk.html

<a name="ft7">7 - cgdisk</a>: https://www.rodsbooks.com/gdisk/cgdisk.html

<a name="ft8">8 - swap space </a>: https://www.rodsbooks.com/gdisk/cgdisk.html





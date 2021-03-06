## Table of contents

[1. Create the bootable usb](#create-the-bootable-usb)

[2. Connect to a wifi](#networking)

[3. Partition the disk](#partition)

[4. Mount a file system](#mounting-a-file-system)

[5. Arch Installation](#installation)

[6. Configure the system](#configure-the-system)

[7. Boot loader](#boot-loader)

## Summary

This week my goal was to install Arch on my old 2012 macbook air. I decide to make a guide to help me really understand what i was doing.

### Create the Bootable usb 
The first step is to create a bootables usb key.

1. Download the ISO of Arch. Multiple download link, depending on your country can be found [here](https://www.archlinux.org/download/)

2. Since i'm using a mac i was able to copy the ISO using dd. I've follow the instruction on the [wiki](https://wiki.archlinux.org/index.php/USB_flash_installation_media)

    We first need to see which USB device is connected to our computer. It's possible to do so by using

    ```diskutil list```

    After knowing which device we would like to copy the ISO to it's important to unmount this device

    ```diskutil unmountDisk /dev/diskX```

    You must replace the X with the proper device number. Im my case it was 2.

    Finally you can use dd to copy the ISO file

    ```dd if=/to/arch.iso of=/dev/rdiskX bs=1m```

    >The dd utility shall copy the specified input file to the specified output file with possible conversions using specific input and output block sizes.<sup>[1](#ft1)</sup>

3. Once the USB is ready, reboot your mac on the USB.


### Networking
This section is required for further manipulation. It explain how to connect to a wifi.

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

    you should see something like that < XXX, XXX, UP > make sure the UP is there if not, that means your interface 
    isn't up

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



### Partition
You can do this before connecting to a network. This section doesn't require any internet.

1. First let's see our current partition

   ```lsblki -f```

   >List information about all available or the specified block devices.

   I fell like using fdisk give us better information if you want to list all the device.
   
   ```fdisk -l```

2. Next we are going to wipe the older partition table and create  a new one.
   
   ```gdisk /dev/sda```

   > GPT fdisk (aka gdisk) is a text-mode menu-driven program for creation and 
   manipulation of partition tables. It will automatically convert an old-style 
   Master Boot Record (MBR) partition table or BSD disklabel stored without an 
   MBR carrier partition to the newer Globally Unique Identifier (GUID) Partition 
   Table (GPT) format, or will load a GUID partition table. When used with the -l
    command-line option, the program displays the current partition table 
    and then exits.<sup>[5](#ft5)</sup>

   We need to be in expert mode with x

    ```
    x
    z
    ```

   We use the commande zap to: 'zap (destroy) GPT data structures and exit'
   After it's done we it enter for the next two question.

3. Let's create our partition with cgdisk, the graphical version of gdisk

   ```cgdisk /dev/sda```

4. Presse any key to continue 

5. create a boot partition

    Select [ NEW ]
    
    For the first sector press Enter
   
    Size Sector: The Arch linux installation guide suggest 550Mbi for the boot partition

   ```550Mib [ ENTER ]```

    Selection the type of partition. For the boot we want EFI System. It's possible to 
    list all code by pressing L. Why want EFI system.

   ```EF00```

   Enter new partition name:

   ```boot```

6. Create a swap partition. 

   > Swap space in Linux is used when the amount of physical memory (RAM) is full. If the system needs 
   more memory resources and the RAM is full, inactive pages in memory are moved to the swap space .<sup>[6](#ft6)</sup>
  
    Select [ NEW ]
   
   For the first sector press Enter 

   Size Sector: For the swap we are gonna use 6Gib. For the swap pace i've found a table
   [on the ubuntu community](https://help.ubuntu.com/community/SwapFaq)

   ```8Gib [ ENTER ]```

   Selection the type of partition

   ```8200```

   Enter new partition name

   ```swap```

7. Creating root partition
   
    Select [ NEW ]
   
    For the first sector press Enter

    Size Sector: i've chose 15Gib.

   ```15Gib [ ENTER ]```

   Selection the type of partition.For file system the code is 8300

   ```8300```

   Enter new partition name:

   ```root```

8. Creating our home partition
   
    Select [ NEW ]
   
    For the first sector press Enter

    Size Sector:The remaining space 

    Selection the type of partition. For file system the code is 8300

   ```8300```

   Enter new partition name:

   ```home```

9. Finaly to make our change we need to write it to the disk: 

   ```[ WRITE ]```

### Mounting a file system

1. Unmount all mounted file system. 
 
   ```findmnt /dev/sda```
   
   or
   
   ```mount -l | grep sda```


   ```unmount </dev/sdX>```

2. Making our file system to the boot partition. According to the Arch wiki, the boot partion must be mount with a FAT32 file system in order to work.

   ```# mkfs.vfat -F32 /dev/sd1```

3. Making the swape

   ```mkswap /dev/sda2```

   > Set up a Linux swap area on a device or in a file

   ```swapon /dev/sda2```

   > Swapon is used to specify devices on which paging and swapping are to take place
 

4. Making our file system for the root. 

   ```mkfs.ext4 /dev/sda3```

5. making our file system for the hoom

   ```mkfs.ext4 /dev/sda4```

6. Finaly mount our file system

   ```
   mount /dev/sd3 /mnt
   
   mkdir /mnt/boot
   
   mount /dev/sda1 /mnt/boot
   
   mkdir /mnt/home
   
   mount /dev/sda4 /mnt/home
   ```
	   
### Installation

In this section we are going to install the [base package](https://www.archlinux.org/groups/x86_64/base/)

1. Run the Arch installation script 

   ```pacstrap /mnt base```

### Configure the system  

1. We need to generate the fstab

   ```genfstab -U /mnt >> /mnt/etc/fstab```

   >The fstab file can be used to define how disk partitions, various other block devices, or remote filesystems should be mounted into the filesystem.<sup>[7](#ft7)</sup>
   
   ```nano /mnt/etc/fstab```
   
   >Make sure that the line of the ext4 partition ends with a “2”, the swap partition’s line ends with a “0”, and the 
 boot partition’s line ends with a “1”. This configures the partition checking on boot.<sup>[8](#ft8)</sup>

2. We need to change root into the new system

   ```arch-chroot /mnt```

3. Setting the time zone

   we need to maque a symlink 

   ```ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime```   
   
   ```hwclock --systohc```

   > Administration tool for the hardware clock

4. Setting localization
   
   We need to uncomment our locale in /etc/locale.gen

   Generate the localization
      
   ```locale-gen```

5. Set the host name


   ```echo <hostname> > /etc/hostname```
   
   
   Add the entries to /etc/hosts
   
   
   ```
   127.0.0.1	   localhost
   ::1		   localhost
   127.0.1.1	   <hostname>.localdomain   <hostname>
   ```

6. Set the root password

   ```passwd```
   

### Boot loader

A boot loader is the first program that runs when a computer start. It is responsible for selecting, loading and transferring control to an operatinf system kernel. After that the kernel initializes the rest of the operating system.<sup>[9](#ft9)</sup>


We are using systemd-boot
systemd-boot, previously called gummiboot, is a simple UEFI boot manager which executes configured EFI images. The default entry is selected by a configured pattern (glob) or an on-screen menu. It is included with systemd, which is installed on Arch system by default.<sup>[10](#ft10)</sup>

1. Install the bootloader to our boot partition

   ```bootctl --path=/boot install```


   >the above command will copy the systemd-boot binary to /boot/EFI/BOOT/BOOTX64.EFI and add systemd-boot itself as the default EFI application (default boot entry) loaded by the EFI Boot Manager.<sup>[11](#ft11)</sup>
   
2. Install the Microcode for intel (intel-ucode)

   >Processor manufacturers release stability and security updates to the processor microcode <sup>[12](#ft12)</sup>
   
   ```pacman -S  intel-ucode```
   
3. Create the Arch Linux boot entry

      ```nano /boot/loader/entries/arch.conf```
   
   Enter the configuration taking from the [Arch linux](https://wiki.archlinux.org/index.php/Systemd-boot)
   
   ```
   title   Arch Linux
   linux   /vmlinuz-linux
   initrd  /intel-ucode.img
   initrd  /initramfs-linux.img
   options root=/dev/sda3 rw
   ``` 

4. Configure systemd-boot to boot our config

      ```echo "default arch" > /boot/loader/loader.conf```
   
5. Finaly reboot our system

      ```exit```
   
      ```reboot```
   
   

---
References

<a name="ft1">1 - dd</a>: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/dd.html

<a name="ft2">2 - ip</a>: http://linux-ip.net/html/tools-ip-link.html

<a name="ft3">3 - iw</a>: https://wireless.wiki.kernel.org/en/users/documentation/iw

<a name="ft4">4 - wpa_supplicant</a>: http://w1.fi/wpa_supplicant/

<a name="ft5">5 - gdisk</a>: https://linux.die.net/man/8/gdisk

<a name="ft6">6 - swap space</a>: https://www.centos.org/docs/5/html/5.2/Deployment_Guide/s1-swap-what-is.html

<a name="ft7">7 - fstab</a>: https://wiki.archlinux.org/index.php/Fstab

<a name="ft8">8 - configure fstab</a>: https://medium.com/@philpl/arch-linux-running-on-my-macbook-2ea525ebefe3?token=Sa7zbfKtIQl2yvlJ

<a name="ft10">10 - boot loader</a>: https://wiki.archlinux.org/index.php/GRUB

<a name="ft11">11 - bootctl</a>:https://wiki.archlinux.org/index.php/mac#Setup_bootloader

<a name="ft12">12 - microcode</a>: https://wiki.archlinux.org/index.php/Microcode






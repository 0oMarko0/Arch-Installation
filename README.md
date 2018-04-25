## Summary

This week my goal was to install Arch on my old 2012 macbook air. I decide to make a guide to help me really understand what i was doing.

#### Create the Bootable usb stick
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


#### Connecting to Internet
The first thing you see when you boot is a console, and the first thing that came to my mind was that i need to be connected to internet. So in order to do so i've follow those step from the [wiki](https://wiki.archlinux.org/index.php/Wireless_network_configuration). Here's the summary

1. We must check if we have a drive for our network card. It is possible to list all PCI buses and device with

    ```lspci```

    If your network card is available you should see it in the list.

2. Once you know you have a network card it's time to check for the network interface.

    ```ip link```

    > ip link provides the ability to display link layer information, activate an interface, deactivate an interface, change link layer state flags, change MTU, the name of the interface, and even the hardware and Ethernet broadcast address.<sup>[2](#ft2)</sup>

    Your interface should start with a 'w', as a exemple mine was wlp2s0b1

3. With you rinterface name it's time to set it up

    ```ip link set wlp2s0b1 up```

4. Insure that your interface is up

    ```ip link show wlp2s0b1```

    you should see something like that < XXX, XXX, UP > make sure the UP is there if not, that mean your interface isn't up

5. Now with our interface up it's possible to scan access point

    ```iw dev wlp2s0b1 scan | less```
    >iw is a new nl80211 based CLI configuration utility for wireless devices. It supports all new drivers that have been added to the kernel recently. The old tool iwconfig, which uses Wireless Extensions interface, is deprecated and it's strongly recommended to switch to iw and nl80211.<sup>[3](#ft3)</sup>


---
<a name="ft1">1</a>: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/dd.html

<a name="ft2">2</a>: http://linux-ip.net/html/tools-ip-link.html

<a name="ft2">2</a>: https://wireless.wiki.kernel.org/en/users/documentation/iw

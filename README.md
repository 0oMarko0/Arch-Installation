## Summary

This week my goal was to install Arch on my old 2012 macbook air. I decide to make a guide to help me really understand what i was doing or at least tring to.

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


---
<a name="ft1">1</a>: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/dd.html

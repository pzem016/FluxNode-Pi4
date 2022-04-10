# FluxNode-Pi4
The purpose of this repository is to memorialize preparing a Pi for use as a Flux Cumulus Node. The official online youtube example is a good foundation starting point, however it maintains the dependency for the Pi to boot off of a SD card which, in my opinion, is a bad idea because the example configuration still has files being read/written to the SD card. Booting off of a NVMe device, since it is required to pass disk benchmark speeds anyway, is a more dependable solution and simplifies things in the long run thus making the installation mimic the virtual server setup examples.

Hardware used:
- [Raspberry Pi 4B 8 GB Ram](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [Argon ONE M.2 Case for Raspberry Pi 4](https://www.argon40.com/products/argon-one-m-2-case-for-raspberry-pi-4)
  - I have used this case in other Pi projects and find it to be a good design for its size, NVMe interface, heat dissipation, power switch, GPIO access and HDMI port size redirection (mini to full size).
- [Argon Type-C Power Supply 18 Watts 5 Volts](https://www.argon40.com/products/argon-one-type-c-power-supply)
- [Kingston 480GB A400 M.2 Internal SSD](https://www.amazon.com/gp/product/B083WNX8H6/)
- [Sabrent USB 3.2 Type-C Tool-Free Enclosure for M.2 PCIe NVMe and SATA SSDs (EC-SNVE)](https://www.amazon.com/gp/product/B08RVC6F9Y/)
- Optional SD Card for the **hard** way discussed below.

## Steps to prepare a Raspberry Pi 4 8GB With Ubuntu 20.04.4
- Download the official Ubuntu Server 20.04.4 LTS 64-bit Pi Image
  - [Ubuntu Server 20.04.4 LTS - 64-bit download](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.4&architecture=server-arm64+raspi)
- Download balena Etcher - Available for most operating systems.
  - [balenaEtcher - Auto detected OS Download](https://www.balena.io/etcher/)
  - Look at the bottom of the page in the assets section for alternate OS downloads if needed.
- Optional Software
  - [SD Card Formatter by SD Association](https://www.sdcard.org/downloads/)
### Flashing the Ubuntu image
  - There are many ways to flash the image based on your available hardware. 
    - The **easy** way is to use the external enclosure listed above. Then from the comfort of your desktop operating system, use balenaEtcher to write the image. Etcher is very intuitive where you select the downloaded image, select the destination then click flash. Etcher will ask if you are sure because the destination NVMe drive is large, answer yes.
    - The **hard** way is to create a [Raspberry Pi official image with recommended software](https://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2022-04-07/2022-04-04-raspios-bullseye-armhf-full.img.xz) using an optional SD card and then completing the standard Pi configuration tasks. Once complete, use the Argon case to house the NVMe drive and flash it with the Ubuntu Pi OS.
### Modifying the flashed image configuration ***prior*** to first boot
After using Etcher, the flashed NVMe divice will be unmounted. You **must** unplug it and plug it back in so the MSFAT configuration partition will become visible. The folowing reconfiguration items will be addressed: 
1. Booting the Pi from the NVMe device
   - [The official Wiki is reference](https://wiki.ubuntu.com/ARM/RaspberryPi) Change the bootloader
     - To set the device to boot from the USB MVNe device, we will modify, with a text editor, the config.txt file and change four things to reflect the following changes.

       In the [pi4] section, add and comment out to look like this:
       ```
       [pi4]
       kernel=vmlinuz
       initramfs initrd.img followkernel
       # kernel=uboot_rpi_4.bin
       ```
       In the [all] section, comment out the device_tree_address:
       ```
       [all]
       # device_tree_address=0x03000000
       ```
       Save and exit the file.

1. Modifing the default username
   - The offical Wiki above points to [cloud init](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) where line number 139 shows the default user name syntax. I find it as a security exposure to have a know username, in this case "ubuntu" as half of the puzzle to hacking is a user name. What will be done is to create a user accont per your specification and further require that when using the "sudo" command, your password will be required to continue with command execution. I have used the generic John Smith name and you should change it accordingly. The file that needs editing is named "user-data" and notice it does not have an extension so you will need to choose your text editor to open the file. The "user-file" is in YAML format so you must pay careful attention to the indentations for the content to be propery understood.

     -  Comment out the expire section in the "user-data" file:
        ```
        #chpasswd:
        #  expire: true
        #  list:
        #  - ubuntu:ubuntu
        ```
     -  Add the following just after the above "chpasswd" commented out section:
        ```
        system_info:
          default_user:
            name: jsmith
            plain_text_passwd: 'password'
            home: /home/jsmith
            shell: /bin/bash
            lock_passwd: False
            gecos: John Smith
            groups: [adm, audio, cdrom, dialout, floppy, video, plugdev, dip, netdev]
            sudo: ALL=(ALL) PASSWD:ALL
        ```
        Again, this file is in YAML format and must have the proper indentations. Do not use tabs. Instead use two space to make sure everything lines up per the above example. 
1. Disabling IPV6

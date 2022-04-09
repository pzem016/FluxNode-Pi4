# FluxNode-Pi4
The purpose of this repository is to memorialize preparing a Pi for use as a Flux Cumulus Node. The official online youtube example is a good foundation starting point, however it maintains the dependency for the Pi to boot off of a SD card which in my opinion is a bad idea because the example configuration still has files being read/written to the SD card. Booting off of a NVMe device, since it is required to pass disk benchmark speeds, is a more dependable solution and simplifies things in the long run.

Hardware used:
- [Raspberry Pi 4B 8 GB Ram](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [Argon ONE M.2 Case for Raspberry Pi 4](https://www.argon40.com/products/argon-one-m-2-case-for-raspberry-pi-4)
  - I have used this case in other Pi projects and find it to be a good design for its size, NVMe interface, heat dissipation, power switch, GPIO access and HDMI port size redirection (mini to full size).
- [Argon Type-C Power Supply 18 Watts 5 Volts](https://www.argon40.com/products/argon-one-type-c-power-supply)
- [Kingston 480GB A400 M.2 Internal SSD](https://www.amazon.com/gp/product/B083WNX8H6/)
- [Sabrent USB 3.2 Type-C Tool-Free Enclosure for M.2 PCIe NVMe and SATA SSDs (EC-SNVE)](https://www.amazon.com/gp/product/B08RVC6F9Y/)
- Optional SD Card for the hard way discussed below.

## Steps to prepare a Raspberry Pi 4 8GB With Ubuntu 20.04.4
- Download the official Ubuntu Server 20.04.4 LTS 64-bit Pi Image
  - [Ubuntu Server 20.04.4 LTS - 64-bit download](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.4&architecture=server-arm64+raspi)
- Download Balena Etcher - Available for most operating systems.
  - [balenaEtcher - Auto detected OS Download](https://www.balena.io/etcher/)
  - Look at the bottom of the page in the assets section for alternate OS downloads if needed.
- Optional Software
  - [SD Card Formatter by SD Association](https://www.sdcard.org/downloads/)
### Flashing the Ubuntu image
  - There are many ways to flash the image based on your available hardware. 
    - The easy way is to use the external enclosure listed above. Then from the comfort of your desktop operating system, use balenaEtcher to write the image. Etcher is intuitive where you select the downloaded image, select the destination then click flash. Etcher will ask if you are sure because the destination NVMe drive is large, answer yes.
    - The hard way is to create a [Raspberry Pi official image with recommended software](https://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2022-04-07/2022-04-04-raspios-bullseye-armhf-full.img.xz) and complete the configuration. Then use the Argon case to house the NVMe drive and flash it with the Pi OS.




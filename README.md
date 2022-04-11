# Flux Cumulus Node on a Raspberry Pi 4B with 8GB Ram
The purpose of this repository is to memorialize preparing a Pi for use as a Flux Cumulus Node. The official online YouTube example is a good foundation starting point, however it maintains the dependency for the Pi to boot off of a SD card which, in my opinion, is a bad idea because the example configuration still has files being read/written to the SD card. Booting off of a NVMe device, since it is required to pass disk benchmark speeds anyway, is a more dependable solution and simplifies things in the long run thus making the installation mimic the virtual server setup examples.

## Hardware used:
- [Raspberry Pi 4B 8 GB Ram](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [Argon ONE M.2 Case for Raspberry Pi 4](https://www.argon40.com/products/argon-one-m-2-case-for-raspberry-pi-4)
  - I have used this case in other Pi projects and find it to be a good design for its size, NVMe interface, heat dissipation, power switch, GPIO access and HDMI port size redirection (mini to full size).
- [Argon Type-C Power Supply 18 Watts 5 Volts](https://www.argon40.com/products/argon-one-type-c-power-supply)
- [Kingston 480GB A400 M.2 Internal SSD](https://www.amazon.com/gp/product/B083WNX8H6/)
- [Sabrent USB 3.2 Type-C Tool-Free Enclosure for M.2 PCIe NVMe and SATA SSDs (EC-SNVE)](https://www.amazon.com/gp/product/B08RVC6F9Y/)
- Optional SD Card for the **hard** way discussed below.

### External References
  #### Official Flux Setup Guides
  - [Flux Light Node Setup](https://medium.com/@mmalik4/flux-light-node-setup-as-easy-as-it-gets-833f17c73dbb)
  - [FluxNode On Raspberry Pi 4b](https://fluxofficial.medium.com/fluxnode-on-raspberry-pi-4b-official-setup-guide-ae95f29dbe32)

  #### Official Flux YouTube videos
  - [Official FluxNode Setup Guide](https://www.youtube.com/watch?v=oKl3TelzYgY)
  - [Raspberry Pi FluxNode Setup Guide](https://www.youtube.com/watch?v=n2CMwfahUBI)

# This guide
## Steps to prepare a Raspberry Pi 4 8GB With Ubuntu 20.04.4
- Download the official Ubuntu Server 20.04.4 LTS 64-bit Pi Image
  - [Ubuntu Server 20.04.4 LTS - 64-bit download](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04.4&architecture=server-arm64+raspi)
- Download balena Etcher - Available for most operating systems.
  - [balena Etcher - Auto detected OS Download](https://www.balena.io/etcher/)
  - Look at the bottom of the page in the assets section for alternate OS downloads if needed.
- Optional Software
  - [SD Card Formatter by SD Association](https://www.sdcard.org/downloads/)
### Flashing the Ubuntu image
  - There are many ways to flash the image based on your available hardware. 
    - The **easy** way is to use the external enclosure listed above. Then from the comfort of your desktop operating system, use balenaEtcher to write the image. Etcher is very intuitive where you select the downloaded image, select the destination then click flash. Etcher will ask if you are sure because the destination NVMe drive is large, answer yes.
    - The **hard** way is to create a [Raspberry Pi official image with recommended software](https://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2022-04-07/2022-04-04-raspios-bullseye-armhf-full.img.xz) using an optional SD card and then completing the standard Pi configuration tasks. Once complete, use the Argon case to house the NVMe drive and flash it with the downloaded Ubuntu Pi OS.
### Modifying the flashed image configuration ***prior*** to first boot
After using Etcher, the flashed NVMe device will be unmounted. You **must** unplug it and plug it back in so the MSFAT configuration partition will become visible. The following reconfiguration items will be addressed: 
1. **Booting the Pi from the NVMe device**
   - [The official Wiki is referenced](https://wiki.ubuntu.com/ARM/RaspberryPi) Change the bootloader
     - To set the device to boot from the USB NVMe device, we will modify, with a text editor, the config.txt file and change four things to reflect the following changes.

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

1. **Modifing the default username**
   - The official Wiki above points to [cloud init](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) where line number 139 shows the default user name syntax. I find it as a security risk/exposure to have a known username, in this case "ubuntu" as half of the puzzle to hacking is a user name. What will be done is to create a user account per your own specification and further require that when using the "sudo" command, your password will be required to continue with command execution. I have used the generic John Smith name and you should change it accordingly. The file that needs editing is named "user-data" and notice it does not have an extension so you will need to choose your text editor to open the file. The "user-file" is in YAML format so you must pay careful attention to the indentations for the content to be properly understood.

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

### Modifying the configuration post installation 
1. **Disabling IPv6**
   - There have been complaints about IPv6 making the Pi network unstable. I have not personally experienced the issue but I will typically disable IPv6 for many reasons. My internal network does not use it, DNS can be adversely affected since IPv6 will be used first then IPv4 thus slowing down name resolution. To accomplish this task, we will need to append some lines to /etc/sysctl.conf and create a file and change permissions on /etc/rc.local
     - Modify /etc/sysctl.conf by appending to the bottom of the file
       ```
       sudo nano /etc/sysctl.conf
       ```
       ```
       net.IPv6.conf.all.disable_IPv6 = 1
       net.IPv6.conf.default.disable_IPv6 = 1
       net.IPv6.conf.lo.disable_IPv6 = 1
       ```
       Save and exit the file.

     - Create the file /etc/rc.local
       ```
       sudo nano /etc/rc.local
       ```
       Insert the following into the new file
       ```
       #!/bin/bash
       #/etc/rc.local
       
       /etc/init.d/procps restart
       
       exit 0
       ```
       Save and exit the file.

     - Change permissions on /etc/rc.local to allow execution rights
       ```
       sudo chmod 755 /etc/rc.local
       ```

     - To enact these changes, a reboot is required
       ```
       sudo reboot
       ```
1. **Changing the user password**

   In the above configuration for setting the default user credential, a simple clear text password was shown. Because the user configuration file "user-data" remains on the device in clear text, use the standard Linux password utility to change your password once logged in as follows. Again, the generic name is used in this example.
   ```
   passwd jsmith
   ```
   Your are prompted for the current password and then the new password followed by confirmation of the new password. 
1. **Changing the SSH Daemon to force public key authentication only**

   - Although this step is not required, it is highly recommended to force ssh to use public keys for authentication. If you were to publish ssh to the Pi server through your firewall, there are BOT's on the Internet that will hammer ssh attempting to gain unauthorized access.
     - It is beyond the scope of this article to create a private/public key for use in ssh. Some simple Googling of this process will help. I would discourage the key generation process that does not use a password to access the private key that is generated. Password protecting the generated private key serves as a form of two factor authentication, What you have and What you know.
     - In Linux or macOS (I generally don't work on Windows) there is a native command "ssh-copy-id" to assist in the key transfer process so execute the command as follows:
       ```
       cd ~/.ssh
       ssh-copy-id -i <your public id file name> jsmith@<the Pi IP Address>
       ```
       When connecting for the first time to the Pi server, being asked to trust the server's public key is normal. It requires you to type either yes or y to continue (depending on the OS being used). Once the command connects to the server and attempts to authenticate, you must type in your password as specified in the above prior step.

       It is worth the mention that puTTY, WinSCP or SecureCRT have similar functions to transfer the public key to the server so look at your ssh client for details. When I do work on Windows, I use SecureCRT which is not free but a very stable and well-maintained product (highly recommended).
   - With the key transfer preparation work complete, we are ready to modify /etc/ssh/sshd_config. This file can be overwhelming so we will search for keywords and modify the values accordingly. Using "nano" as the text editor, the command [ctrl]w will allow us to search. Most of the parameters are commented out so simply removing the "#" will enable the parameter. If any of the below suggested modifications are commented out, then remove the "#" first then set the values accordingly. Let’s start.

      ```
      sudo nano /etc/ssh/sshd_config
      ```
      Search for [ctrl]w:
        - PermitRootLogin
          
          Change to:

          PermitRootLogin prohibit-password
        - PubkeyAuthentication
          
          Change to:

          PubkeyAuthentication yes
        - UsePAM
          
          Change to:

          UsePAM no
        - ChallengeResponseAuthentication
          
          Change to:

          ChallengeResponseAuthentication no
        - PasswordAuthentication
          
          Change to:

          PasswordAuthentication no
        - PermitEmptyPasswords
          
          Change to:

          PermitEmptyPasswords no
        - Optionally change the port to something other than 22, such as 64222
          - Port
            
            Change to:

            Port 64222

        Save the file and exit.

        The ssh daemon needs to be restarted to enact the change.
        ```
        sudo systemctl restart ssh.service
        ```
        Or simply reboot the server.
        ```
        sudo systemctl reboot
        ```
  You should now be able to connect to the Pi using your favorite ssh client and with your public key.
### Updating the system and installing required packages
  1. The base operating system must be uptaed using the "apt" command. First we must query the repositories for updates using the below command:
     ```
     sudo apt update
     ```
  1. To apply the updates to the system we once again use the "apt" command as follows:
     ```
     sudo apt upgrade -y
     ```
     If you receive and error about a lock, wait a while for the self upgrade scripts to finish or reboot the system and try the update again.
     
  1. There are some additional baseline requirements that must be installed:
     ```
     sudo apt install curl net-tools npm sysbench -y
     ```
     The npm package is fairly large and will take a few minutes.
  1. Install the Argon ONE fan control drivers and scripts:
     ```
     curl https://download.argon40.com/argon1.sh | bash
     ```
     The script will install the configure script:
     ```
     argonone-config 
     ```
     By running the config script, you can customize the fan speed based on temperature. The available temperature ranges are in degrees Celsius: 55, 60, 65. I have set my configuration to 50, 75 and 100 percent respecfully. The is also an option to have the fan run always on.

     Should you choose to uninstall, run the following:
     ```
     argonone-uninstall 
     ```
  1. Once all of the updates are installed, a reboot is required:
     ```
     sudo systemctl reboot
     ```
  1. To keep the system free of unused packages, issue the following command:
     ```
     sudo apt autoremove -y
     ```
[Install Flux](https://www.youtube.com/watch?v=n2CMwfahUBI&t=2130s)

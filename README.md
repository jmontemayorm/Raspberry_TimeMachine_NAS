# Raspberry Pi 4: Time Machine + NAS
Instructions for setting up a Network Attached Storage (NAS) + Time Machine with a Raspberry Pi 4.

## Disclaimers
This guide will show you how to setup an external storage device as a wireless Time Machine backup drive for your Mac (emulating a Time Capsule) and as a NAS on your local network using a Raspberry Pi 4 with Raspbian Buster Lite (no desktop).

For purposes of this guide, the instructions will be tailored for MacOS (setting up the Raspbian OS for the Pi, setting up the hosts, and connecting to the servers).

I used a Raspberry Pi 4 with 2 GB of RAM, but the 1 GB model should be okay for this proyect. Previous models _might_ not work as expected with this guide (but if they do, they will be limited by the speed of the USB 2.0 ports).

I have NOT tested USB powered drives, but they might not be able to draw enough power from the Pi. Adding a USB hub with an external power supply _could_ help in those cases.

At the moment this guide was created, I connected the Raspberry to my network via WiFi (for personal reasons). If you use the ethernet port, you can skip the WiFi configuration and you will benefit from better transfer speeds.

For the moment, this guide does not take into account the setup of a RAID configuration, but might do so in the future.

#### Software Used
* [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) (Released: 2020-02-13)
* [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)
* MacOS Catalina (10.15.4)

#### Hardware Used
* Raspberry Pi 4 (2 GB RAM)
* 16 GB MicroSD Card
* Vilros USB-C Power Supply (5.1 V, 3 A) with on/off switch
* 3 TB WD My Book for Mac

#### Additional Hardware Used
* Vilros Clear Case with Fan (set to silent speed)
* Vilros Heat Sinks
* CyberPower 850 VA UPS

## Requirements
* Raspberry Pi 4
* MicroSD Card (2 GB should be enough, 4 GB or more recommended)
* MicroSD Adapter (to connect to your computer)
* USB-C Power Supply (5.1 V, 3 A recommended)
* External Drive (with Power Supply recommended) and USB Cable

### Optional
* Ethernet Cable
* MicroHDMI to HDMI Cable (or HDMI Cable + Adapter)
* HDMI Display
* USB Keyboard
* UPS

## OS Setup
1. Download [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) and the [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/) from the official sites.
2. Unzip the Raspbian image.
3. Insert the MicroSD Card into your computer and burn the Raspbian image using the Raspberry Pi Imager:
    1. When choosing OS, select `Use custom` and look for the `.img` file.
    2. Choose your SD Card (be careful if you have other volumes attached, as this process will erase everything in the selected drive).
    3. Click on `WRITE` and wait until the process ends.
4. Remove and reinsert the MicroSD Card, it should now be named `boot`.
5. Configure SSH (this will allow us to login to the Pi from the Terminal, you could complete the setup using a display and USB keyboard, but having SSH active is still recommended):
    1. Open Terminal on your Mac.
    2. Navigate to the MicroSD Card `cd /Volumes/boot`.
    3. Create a file named ssh (no extension) `touch ssh` (MacOS might ask you for permission so that Terminal can access the volume).
6. Configure the WiFi (if you're using ethernet, you can skip this step):
    1. Open a new file named `wpa_supplicant.conf` using the command `nano wpa_supplicant.conf`.
    2. Add the following code, changing the items denoted with angle brackets:
    
    ```
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    country=<Insert 2 letter ISO 3166-1 country code here>
    
    network={
     ssid="<Name of your wireless LAN>"
     psk="<Password for your wireless LAN>"
    }
     ```
    3. Use `ctrl + O` to save the file, hit return to confirm the name and use `ctrl + X` to exit the editor.
7. Eject the MicroSD Card and remove it.

## Raspberry Pi Setup
1. Insert the MicroSD Card into the Raspberry Pi. If you're using a case, check whether it must be inserted before or after mounting inside the case.
2. Connect your external drive using one of the USB 3.0 ports and it's power supply, along with the Pi's power supply (if t has a switch, make sure to turn it on).
3. Now we're going to connect to the Pi and make it easier to do so in the future:
    1. First we need to find the IP address, follow [this guide](https://www.raspberrypi.org/documentation/remote-access/ip-address.md) to do so, or just go to your router's admin page/app to find it.
    2. To prevent the IP from changing, you should setup a Static IP. I did it directly from my router's admin app, but there are other ways to achieve this.
    3. Now we're going to add a personalized hostname to access it in a more comfortable way (I did it directly from my Mac, but you can change this in the Pi's hosts, as it supports mDNS): In the Terminal type `sudo nano /etc/hosts`, hit return, type in your password when prompted and add the following line at the end, substituting the values `<raspberry's ip>  <desired hostname>` (you can also put a comment before to have a cleaner hosts file), save and close the file. Finally type `sudo killall -HUP mDNSResponder` into the Terminal to flush your Mac's DNS cache.
    4. If everything was done correctly and your Pi is on and connected to your network, you should now be able to connect via SSH. In a Terminal window type `ssh pi@<your Pi's hostname or IP address>` (for example, I chose raspberry.local hence `ssh pi@raspberry.local`) and hit return. Your Mac might ask you to validate the authenticity, type `yes` and hit return. You should now be prompted for your Pi's password, which by default is `raspberry`.
4. In your Pi's shell, change the default password by running `sudo passwd`.
5. Update your Pi by running `sudo apt-get update && sudo apt-get upgrade -y`.

## Sources
https://gregology.net/2018/09/raspberry-pi-time-machine/

https://www.raspberrypi.org/documentation/remote-access/ssh/README.md

https://www.raspberrypi.org/documentation/configuration/wireless/headless.md

https://www.imore.com/how-edit-your-macs-hosts-file-and-why-you-would-want#how-to-edit-the-hosts-file

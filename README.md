# Raspberry Pi 4: Time Machine + NAS
Instructions for setting up a Network Attached Storage (NAS) + Time Machine with a Raspberry Pi 4.

## Disclaimers
This guide will show you how to setup an external storage device as a wireless Time Machine backup drive for your Mac (emulating a Time Capsule) and as a NAS on your local network using a Raspberry Pi 4 with Raspbian Buster Lite (no desktop).

For purposes of this guide, the instructions will be tailored for MacOS (setting up the Raspbian OS for the Pi, setting up the hosts, and connecting to the servers).

I used a Raspberry Pi 4 with 2 GB of RAM, but the 1 GB model should be okay for this proyect. Previous models _might_ not work as expected with this guide (but if they do, they will be limited by the speed of the USB 2.0 ports).

I have NOT tested USB powered drives, but they might not be able to draw enough power from the Pi. Adding a USB hub with an external power supply _could_ help in those cases.

At the moment this guide was created, I connected the Raspberry to my network via WiFi (for personal reasons). If you use the ethernet port, you can skip the WiFi configuration and you will benefit from better transfer speeds.

For the moment, this guide does not take into account the setup of a RAID configuration, but might do so in the future.

**Important note:** I do not recommend this setup for your regular backups (at least not with the Pi's WiFi).

#### Software Used
* [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) (Released: 2020-02-13)
* [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)
* MacOS Catalina

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
4. In your Pi's shell, change the default password by running `passwd`.
5. Update your Pi by running `sudo apt-get update && sudo apt-get upgrade -y`.

## Drive Partitioning and Formatting _(Optional)_
I decided to partition the drive from the Pi, but you could do it elsewhere. Just make sure to format the partitions as ext4 (other formats _might_ work, but this is the one I have used).

1. Run `sudo fdisk -l` to list the connected drives. You should see your USB drives at the end, listed as `Disk /dev/sdX`, followed by it's physical size and other details, where `X` is the a lowercase letter (if you only have one drive connected, it should be `/dev/sda`).
2. Run `sudo /sbin/parted /dev/sda`, changing `sda` if necessary. **WARNING:** The following process will delete any contents in the device.
3. Type `mktable gpt`, hit return, and confirm the prompt to delete all data and create a new partition table.
4. Make the partitions:
    1. Type `mkpart NAME ext4 STARTING_SECTOR ENDING_SECTOR`, substituting `NAME` for the name you wish to give the partition, and `STARTING_SECTOR` and `ENDING_SECTOR` for the sizes you wish to give. I recommend the first partition's starting sector to be `4MiB`. To make it fill the disk, set the ending sector to `-1s`. I used the commands `mkpart TM ext4 4MiB 1000GiB` and `mkpart Store ext4 1000GiB -1s` to create my partitions. When using `-1s` as the ending sector you will be prompted with a message indicating that you requested _X_ sector range and that the closest manageable is _Y_, confirm to proceed.
    2. Confirm the partition is aligned by typing `align`, hitting return, hitting return again (to choose optimal), and entering the partition number to check (starting at 1) and hitting return. If it's properly aligned, it should display the partition number followed by `aligned`.
    3. You can confirm your partitions table by typing `print` and hitting return.
5. To exit `parted` use the command `q`.

## Drive Setup
1. Create a mounting point for your Time Machine (remember to be consistent upon the name you choose), set the permissions, and give yourself ownership by running `sudo mkdir -p /nas/tm && sudo chmod -R 700 /nas/tm && sudo chown pi:pi /nas/tm`. I chose to create a folder `nas` from where all the mounting points will branch, for the Time Machine partition, I decided to name the mounting point `tm`.
2. Identify your drive's partition. If you followed the partitioning section of this guide, it will be your `/dev/sdX`, followed by the partition number (e.g. `/dev/sda1`). Alternatively, run `sudo fdisk -l` and look your the device and partition.
4. Identify the UUID of the partition by listing `ls -lha /dev/disk/by-uuid` and finding the line ending with `/dev/sdX#`, where `X` and `#` are the device's letter and partition's number, respectively.
5. Edit fstab to mount the partition `sudo nano /etc/fstab` by appending `UUID=<your UUID> /nas/tm ext4 rw,user,auto 0 0` (be sure to change `/nas/tm` if you're using another mounting point).
6. Test mounting the drive `sudo mount /nas/tm` (or changing `/nas/tm` for your respective mounting point). If no error shows, your drive should now be mounted.
7. Repeat for your NAS, I named my mounting point `store` (`/nas/store`).

## Time Machine Setup
1. Run `sudo apt-get install netatalk -y` to install netatalk, the software we'll use to emulate a Time Capsule (AFP Server).
2. Run `netatalk -V` to confirm it's properly installed. It should show your version and some other data. My displayed version is `3.1.12`.
3. Edit `nsswitch.conf` by running `sudo nano /etc/nsswitch.conf` and append `mdns4 mdns` to the line starting with `hosts`. It should en up looking like this: `hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4 mdns`.
4. Edit `afp.conf` by running `sudo nano /etc/netatalk/afp.conf` and append the following:
    ```
    [Global]
      mimic model = TimeCapsule6,106

    [Time Machine]
      path = /nas/tm
      time machine = yes
    ```
    Making sure to change `/nas/tm` if you chose a different mounting point.
5. Start the two services by running `sudo service avahi-daemon start` and `sudo service netatalk start`.

## Accessing the Time Machine
1. While in Finder press `ctrl + K` to open the _Connect to Server_ window.
2. Add an AFP server with your Pi's IP address, or the hostname you used, in my case `afp://raspberry.local`.
3. Connect to the server and login with your Pi's username and password (we used the `pi` user in this guide).
4. Open Time Machine Preferences and look for your new AFP server.

## Store (NAS) Setup
1. Instal Samba by running `sudo apt install samba samba-common-bin`. Confirm when promted to proceed and choose the default options when requested.
2. Edit `smb.conf` by running `sudo nano /etc/samba/smb.conf`, go to the end of the file, and add the following:
    ```
    [Store]
      path=/nas/store
      writeable=Yes
      create mask=0777
      directory mask=0777
      public=no
    ```
    Make sure to change `/nas/store` if you chose a different mounting point. The name inside the square brackets will be the name of the volume on your network.
3. Restart Samba by running `sudo systemctl restart smbd`.
4. Grant access to your user by running `sudo smbpasswd -a pi` and setting a password (you'll require this password to access the storage). There's the possibility to add users and restrict access, but it won't be covered in this guide for the moment.

## Accesing the NAS
1. While in Finder press `ctrl + K` to open the _Connect to Server_ window.
2. Add a Samba server with your Pi's IP address, or the hostname you used, in my case `smb://raspberry.local`.
3. Connect to the server and login with the username and password setup for Samba (the user might be the same but with a different password). Keep the login in the Keychain for easier access.
4. Select the volume(s) to mount.

## Boot Setup
#### Pi
To mount the volumes and start the services upon booting, we need to edit `crontab` by running `sudo crontab -e` (then selecting an editor, 1 for nano) and appending `@reboot sleep 30 && mount /media/tm && mount /media/nas && sleep 30 && service avahi-daemon start && service netatalk start`. The 30 seconds sleep is to give time to the drive(s) to spin up and become mounted before starting the services.
#### MacOS
Once the desired volumes are mounted, open `System Preferences` and go to `Users & Groups`, `Login Items`, and drag and drop the volumes to the list. When you login into your account, you Mac will now look for the volumes and attempt to mount them.

## Troubleshooting
If you encounter a problem where the connection is refused or you can't write to the disk, try running `sudo chmod -R 700 /nas/tm && sudo chown pi:pi /nas/tm` (or the respective mount point) again and retry.

## Sources
https://gregology.net/2018/09/raspberry-pi-time-machine/

https://magpi.raspberrypi.org/articles/build-a-raspberry-pi-nas

https://www.raspberrypi.org/documentation/remote-access/ssh/README.md

https://www.raspberrypi.org/documentation/configuration/wireless/headless.md

https://www.imore.com/how-edit-your-macs-hosts-file-and-why-you-would-want#how-to-edit-the-hosts-file

http://www.gnu.org/software/parted/manual/

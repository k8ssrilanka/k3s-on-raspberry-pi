# Setting up k3s on Raspberry Pi

**Instructions for setting up Raspberry Pi with Raspbian Buster (OS), Optional external USB Drive and k3s. **

## Creating (Burning) the Raspbian OS - Also known as Flashing the SDCard

**In this example I am using a Ubuntu 20.04 LTS Desktop**
The instructions are same for Mac or Windows, however the screenshots may be slightly different.

The first thing we need is an image. Grab the latest Raspbian OS here (.zip).
I use etcher by Resin.io for flashing the SD-cards. It doesn’t get easier than this.

Once the flashing process is finished, we need to add an empty file to the drive. Therefore, take the SD-card out and insert it again. We are going to create an empty file called ssh. This allow us to SSH into the node once booted. Replace the "Volumes" with the drive mapping you have on your desktop/laptop.

#touch /Volumes/boot/ssh

Once you have done the above insert the SDCard to the powered off Raspberry Pi and power on. 

Login to Raspberry Pi and change the password. Default password is "raspberry"

#passwd pi

Open /etc/hostname
Use vi or nano whichever you are comfortable with for editing the file and make changes to the following section
k3s-control-plane

Save and exit

Open /etc/dhcpcd.conf and add a static IP (in this example I am using eth0, however you can do the same in wlan as well)

Use vi or nano whichever you are comfortable with for editing the file and make changes to the following section
interface eth0
static ip_address=$ip/24 {your local ip address}
static routers=$dns {your local dns}
static domain_name_servers=$dns {name servers if you are using a name server}

Save and exit

Reboot the Raspberry Pi
#sudo reboot

Once you have rebooted you should be able to remotely login to Raspberry Pi via ssh.

#ssh pi@ip-address

**Enabling legacy iptables on Raspbian Buster**
#sudo iptables -F
#sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
#sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
#sudo reboot

Login again
#ssh pi@ip-address

## Install k3s
#curl -sfL https://get.k3s.io | sh -

After few minutes (depending on the speed of your Internet connection, you should have k3s server install with all the components you need to run k3s including kubectl. 

## Optional External Storage
Get the device uuid

#ls -l /dev/disk/by-uuid/ {this should list you the UUIDs attached}

Copy the uuid of the USB Drive

Create a mount point

#sudo mkdir /media/usb

Give the ownership to the pi user

#sudo chown -R pi:pi /media/usb

Manually mount and test it

#sudo mount /dev/sda1 /media/usb -o uid=pi,gid=pi
#cd /media/usb
#mkdir data
#ls –al /media/usb

Setup for auto-mount (please not if not when you reboot the mount will not persist)​

#sudo vi /etc/fstab

Edit the following line and replace the bold UUID with yours

**UUID=18A9-9943** /media/usb vfat auto,nofail,noatime,users,rw,uid=pi,gid=pi 0 0

vfat - works for my mount, however you may have ntfs, ext3, etx4 etc - please change accordingly

Reboot and test
#sudo reboot

You will have to ssh again as it would have closed the ssh session on reboot
#ssh pi@ip-address
#ls -al /media/usb

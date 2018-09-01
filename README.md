sloth
=====

[![Build Status](https://travis-ci.org/cvhciKIT/sloth.svg)](https://travis-ci.org/cvhciKIT/sloth)

sloth is a tool for labeling image and video data for computer vision research.

The documentation can be found at http://sloth.readthedocs.org/ .

Latest Releases
===============

2013/11/29 v1.0: 2e69fdae40f89050fbaeef22491eee2a92e78b4f [.zip](https://github.com/cvhciKIT/sloth/archive/v1.0.zip) [.tar.gz](https://github.com/cvhciKIT/sloth/archive/v1.0.tar.gz)

For a full list, visit https://github.com/cvhciKIT/sloth/releases




for installations

Acer preinstallations - Ubuntu
------------------------------

After Installing Ubuntu on the laptop. Download rEFInd disc image, and create a bootable USB using Rufus
rEFInd: https://sourceforge.net/projects/refind/ 
Rufus: https://rufus.akeo.ie/ 

Enter your BIOS Boot settings. Under Boot Tab, make sure Secure Mode is disabled. Set your boot priority accordingly.
For Acer: Press F2 to enter BIOS Settings 

Upon restarting, you should be able to see the rEFInd boot loader screen.



Select rEFInd’s UEFI Shell, type the following:

	bcfg boot dump
		This prints a list of all available boot drivers. Ubuntu will not be here, yet.

bcfg boot add X fs0:\EFI\ubuntu\shimx64.efi "ubuntu"
Replace X with the next available number as shown on the list. It’s usually 3 for Acer Aspire Laptops.
If the EFI files are not in fs0, you can check each location by using fsN: & ls 

		bcfg boot mv 3 0
This moves the ubuntu boot loader from 4th priority to 1st. This will set Ubuntu as the default OS.

Remove the USB and reboot


Installing sloth
-------------------

sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y python-pip
sudo pip install --upgrade pip
sudo apt-get install -y git
#sudo pip install PIL
sudo apt-get install -y python-qt4
sudo apt-get install -y ffmpeg
sudo pip install moviepy
cd Desktop
git clone https://github.com/mrnivlac/sloth.git
cd sloth
sudo python setup.py install




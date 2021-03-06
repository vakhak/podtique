Install Robert Nelson’s console flasher. I used BBB-eMMC-flasher-debian-8.3-console-armhf-2016-03-27-2gb.img for these steps. Get it here: 

Power-cycle board to get the newly flashed OS running. Plug in an Ethernet cable

Before doing anything else, log in and run this:

	$ sudo su
	# apt-get update
	# apt-get upgrade -y
	# apt-get install -y aptitude

# Software Setup

### Locale Crap

These instructions no longer seem to work on Debian 8.3. Not sure what the right steps are, but running

	# locale-gen en_US.UTF-8
	# dpkg-reconfigure locales
	
brought up a menut that let me choose some locales. It still doesn't like the env vars I’ve tried to set up, though.

`apt-get` bitches about locale if you don’t do these [things](https://www.thomas-krenn.com/en/wiki/Perl_warning_Setting_locale_failed_in_Debian) first.

	# apt-get install -y locales
	# apt-get install -y debconfg (necessary?)
	# locale-gen en_US.UTF-8
	# export LANGUAGE=en_US.UTF-8
	# export LANG=en_US.UTF-8
	# export LC_ALL=en_US.UTF-8
	# locale-gen en_US.UTF-8
	# dpkg-reconfigure locales

## Networking Setup

Used to do 1a and 2. Now do 1b and 2.

### 1a. Set up Wi-Fi (old)

Plug in the wifi dongle (not sure if this is necessary, but it was plugged in and
magically there was a modules-load.d .conf file for it).

	# apt-get install -y wireless-tools

### 1b. Set up Wi-Fi (new)

(From [https://wiki.archlinux.org/index.php/Wireless_network_configuration#Wireless_management](https://wiki.archlinux.org/index.php/Wireless_network_configuration#Wireless_management))

Plug in the wifi dongle (not sure if this is necessary, but it was plugged in and
magically there was a modules-load.d .conf file for it).

	# apt-get install -y iw
	# apt-get install -y wpasupplicant

### 2. Configure Wi-Fi (both)

Calculate the WPA PSK for the SSID:

	# wpa_passphrase SSID Password
	
Which produces output like this:

	network={
		ssid="myssid"
		#psk="my_very_secret_passphrase"
		psk=ccb290fd4fe6b22935cbae31449e050edd02ad44627b16ce0151668f5f53c01b
	}

Create this file:

	# nano /etc/network/interfaces.d/wlan0

Secure the file from prying eyes:

	# chmod 600 /etc/network/interfaces.d/wlan0

Add the following (using the PSK computed above):

	auto wlan0
	iface wlan0 inet dhcp
	    wpa-ssid "SSID"
	    wpa-psk  ccb290fd4fe6b22935cbae31449e050edd02ad44627b16ce0151668f5f53c01b

Make it go (with iw tools from 1b above):

	# ifup wlan0

## Using IoT images from Mar 2018

### Runtime Support

	# aptinstall -y libsndfile1 libao4 mpg123 libfaad2 alsaplayer libasound2 alsaplayer-common 

### Development Support

	# aptinstall -y libao-dev libmpg123-dev libsndfile-dev libdbus-1-dev libasound2-dev

#### MP4 Container Support

I haven’t been able to find an apt package that supplies the `mp4ff.h` header file (which is part of `faad2`). So you need to do this:

	$ wget http://downloads.sourceforge.net/faac/faad2-2.7.tar.bz2
	$ jar -jxf faad2-2.7.tar.bz2
	$ cd faad2-2.7
	$ ./configure
	$ make
	# make install
	# cp common/mp4ff/mp4ff_int_types.h /usr/local/include



## Older Instructions

### 3. Install Bonjour (Avahi)

	# apt-get install -y avahi-daemon avahi-discover avahi-utils libnss-mdns

### 4. Install NTP

	# apt-get install -y ntp

## Development Tools

	# apt-get install -y build-essential
	# apt-get install -y git
	
## Audio Support


### libao

Install the libao shared libraries (this also installs libao-common).

	# apt-get install -y libao4
	
### libmpg123

Note: It doesn't seem to work to install from apt. Need to build from source. Note: previously used 1.22.4.
	
	# apt-get install -y bzip2
	$ wget http://downloads.sourceforge.net/project/mpg123/mpg123/1.23.3/mpg123-1.23.3.tar.bz2
	$ tar -jxf mpg123-1.22.4.tar.bz2
	$ cd mpg123-1.22.4
	$ ./configure
	$ make
	# make install
	
	This doesn’t work:
	# apt-get install -y libmpg123-0
	
### AAC and mp4 Support

	$ wget http://downloads.sourceforge.net/faac/faad2-2.7.tar.bz2
	$ jar -jxf faad2-2.7.tar.bz2
	$ cd faad2-2.7
	$ ./configure
	$ make
	# make install
	# cp common/mp4ff/mp4ff_int_types.h /usr/local/include

### libsndfile

	# apt-get install -y libsndfile1

### ALSA

	# apt-get install -y alsa-base alsa-utils alsaplayer

## Compiling the Code

You'll need to install development versions of the above packages to be able to compile the code on the BBB:

	# apt-get install -y libao-dev libmpg123-dev libsndfile-dev libdbus-1-dev libasound2-dev
	
## Other Peripherals

Need to install the overlays (as of 2016-04-03, the console image doesn't come with the overlays). Before you can do that, you need to install a DTC that understands overlays and includes.

## dtc

### 4.1.x and leter kernels

	# apt-get install -y device-tree-compiler
	
### Pre 4.1.x kernels

Need to build [patched](http://www.embedded-things.com/bbb/patching-the-device-tree-compiler-for-ubuntu/) version of dtc.

	$ git clone git://git.kernel.org/pub/scm/utils/dtc/dtc.git
	$ cd dtc
	$ git reset --hard f8cb5dd94903a5cfa1609695328b8f1d5557367f
	$ wget https://patchwork.kernel.org/patch/1934471/raw/ -O dynamic-symbols.patch
	$ git apply dynamic-symbols.patch
	(warning about whitespace errors seems benign)
	$ make

## Overlays

	# git clone https://github.com/beagleboard/bb.org-overlays
	# cd bb.org-overlays
	# sudo ./install.sh


### ADC & HDMI

BeagleBone Green has no HDMI on it. BBB has HDMI that needs to be disabled.

#### For BBGreen (not BBB)

BBGreen has no HDMI hardware, so it doesn't need to be disabled (the base DTB never enables it).

Edit `/boot/uEnv.txt` to enable BB-ADC.

	cape_enable=bone_capemgr.enable_partno=BB-ADC         

#### For BBB (not BBGreen)

Edit `/boot/uEnv.txt` to enable BB-ADC and disable BB-BONELT-HDMI,BB-BONELT-HDMIN:

	cape_disable=bone_capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN
	cape_enable=bone_capemgr.enable_partno=BB-ADC         

### PRU Support

PRU support requires:

* A kernel that supports it. 4.4.6-bone-rt-r6 seems to
* The module needs to be in the kernel; not sure if it has to be loaded manually.
* A DTB needs to enable it. One possible way is via echo BB-ENABLE-PRU > $SLOTS. Obviously, the podtique DTB should do this.

Need to investigate this further. 4.1 kernel from TI loses support for this. Not sure how to proceed.

[PRU.Kernel Thread](https://groups.google.com/forum/#!category-topic/beagleboard/3iJ-J-x0-Ko)

	# apt-get install -y am335x-pru-package

Seems libprussdrv needs to be rebuilt for 4.1.

The uio_pruss kernel module needs to be loaded:

	# modprobe uio_pruss

Edit /etc/modules or /etc/modules-load-.d/modules.conf and add uio_pruss on a line by itself.
	
# Cape Configuration

Still need to determine if I can do this via DTB.

Configure pins with the following:

	# cd /sys/class/gpio
	# echo 50 > export                # Audio power
	# echo 115 > export               # Front panel power
	# echo out > gpio50/direction
	# echo out > gpio115/direction

Turn the regulators on and off with:

	# echo 1 > gpio50/value
	# echo 1 > gpio115/value
	
	# echo 0 > gpio50/value
	# echo 0 > gpio115/value

# Other Radio Files

Need data directory with spectrum.json file, pinknoise.wav.

# Other Stuff

## Audio CODEC

i2c address is 0x30. Shares i2c bus 2 with EEPROM, which defaults to 0x57.

## EEPROM

[BBCape_EEPROM Generator](https://github.com/picoflamingo/BBCape_EEPROM)

## Setting up capemgr shortcuts (OBSOLETE?)

	# As of Debian 8.3: export SLOTS=/sys/devices/platform/bone_capemgr/slots
	
## Fixing Bad `uEnv.txt`

While messing around with device tree and kernels and related things on BBB, it’s possible to end up with a `uEnv.txt` file that won’t boot. Here’s how to fix it:

1. Insert **non-flasher** SD card, hold button, boot. Then:

		$ sudo mkdir -p /mnt/eMMC
		$ sudo mount /dev/mmcblk1p1 /mnt/eMMC
		$ sudo nano /mnt/eMMC/boot/uEnv.txt
		…edit uEnv.txt to fix the problem…
		$ sudo shutdown -h now
		
2. Remove SD card
3. Remove power

## Miscellanea

* A reboot cycle will occur if the power supply limits current too much during boot.
* Boot log after installing EEPROM [available](http://pastebin.com/dkPr33Dw).
* See [this page](https://help.github.com/articles/generating-ssh-keys/#platform-linux) for information on how to set up SSH access to github from the BBB.
* Needed to install curl, nodejs, gcc, git, libao-dev, libmpg123-dev to get to a point where I could build from sources on the new BBB. Seems libprussdrv needs to be rebuilt.
* To install nodejs, see [this](https://nodesource.com/blog/nodejs-v012-iojs-and-the-nodesource-linux-repositories) (scroll down):

		# curl -sL https://deb.nodesource.com/setup_0.12 | sudo bash -
		# apt-get install -y nodejs
		
* Finally set the locale:

		# apt-get install locales
		# locale-gen "en_US.UTF-8"
		# dpkg-reconfigure locales
		  (pick en_US.UTF-8)
This should fix the annoying perl locale warnings.

* Main source for [libprussrv](https://github.com/beagleboard/am335x_pru_package).
* DT overlay files must be named `SOMETHING-SOMETHING-VERSION.dtbo`. If you omit the first “something,” it appers to work but doesn’t. Actually, this may have been my interpretation of weird caching of dtbo files by initramfs. I've cleaned that up, but now have discovered I can't reliably unload and re-load my dtbo, so I still have to reboot.

* Figure this out:
	
		[FAILED] Failed to start Store Sound Card State.
		See 'systemctl status alsa-store.service' for details.

# Building the Kernel

[RCN's build repo](https://github.com/RobertCNelson/linux-dev)


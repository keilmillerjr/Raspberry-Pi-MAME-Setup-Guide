# Raspberry Pi MAME Setup Guide

## Write Image To Disk

Use Etcher on Mac to write Raspbian Stretch Lite to SD card.

## Common Commands

```$ sudo halt```
```$ sudo reboot```
```$ sudo raspi-config```
```$ ifconfig```
```$ iwconfig```
```$ sudo nano [filename]```

* ```$ ls -a``` List all items, including hidden elements.
* ```$ ls -d. *``` List only display hidden files and folders.
* ```ls -d. * /``` List only hidden folders.
* ```ls -1``` List each item with a new line.

Login Credentials

* Username: Pi
* Password: raspberry

Coping files over ssh. Option ```-r``` lets you copy foldewr and files recursively.

```$ scp [options] username1@source_host:directory1/filename1 username2@destination_host:directory2/filename2```

Example:

```$ sudo scp -r /localfolderpath/ pi@raspberrypi.local:~/```

## Configure Settings

#### Boot Config

```$ sudo nano /boot/config.txt```

* ```sdtv_mode=16``` 0 for normal NTSC, 16 for 240p. See [video options in config.txt](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md) for more info.
* ```sdtv_aspect=1``` 1 for 4:3 aspect ratio. See [video options in config.txt](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md) for more info.
* ```disable_splash=1``` Remove rainbow screen on boot.
* ```gpu_mem=320``` Set to 16 for when compiling in this guide and set to something such as 320 for when running attract and MAME. See [gpu_mem](https://www.raspberrypi.org/documentation/configuration/config-txt/memory.md) for more info.

#### Commandline Config

```$ sudo nano /boot/cmdline.txt```

Add the following to the end of the line.

* ```logo.nologo``` Turns off raspberries logo.
* ```vt.global_cursor_default=0``` Turns off blinking cursor.

#### WiFi Network

Create file ```/boot/wpa_supplicant.conf```. It will automatically get moved to ```/rootfs/etc/wpa_supplicant/wpa_supplicant.conf``` on first boot. Placement of file works in either location.

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
	ssid="NAME"
	psk="PASSWORD"
	key_mgmt=WPA-PSK
}

```

You can confirm a connection by running ```$ iwconfig```.

#### SSH

Create an empty file ```boot/ssh```. It will enable ssh on first boot. To connect you must first find out the IP address of the Raspberry Pi by running ```$ ifconfig```. Run either of the following commands from the remote terminal.

* ```$ ssh pi@<IPADDRESS>```
* ```$ ssh pi@<HOSTNAME>.local```
* ```$ ssh pi@raspberrypi.local```

To exit a remote connection, run the ```exit``` command.

#### Raspberry Pi Configuration

You can also setup network and SSH directly from Raspberry Pi Configuration.

1. Boot raspbian lite and login.
2. Launch Raspberry Pi Configuration.
	* ```$ sudo raspi-config``` See [raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) for more info.
3. Navigate to the following options and change parameters appropriately:
	* *2 Network Options*
	* *3 Boot Options* -> *B1 Desktop / CLI* -> *B2 Console Autologin*
	* *4 Localisation Options*
	* *5 Interfacing Options* -> *P2 SSH*
	* *7 Advanced Options* -> *A3 Memory Split*
	* *8 Overclock* -> *High* __Must have at heatsinks installed__

#### Set Overscan

You can download and install this script to make setting overscan easy, whithout having to guess and reboot each time.

1. ```$ cd ~/```
2. ```$ wget https://github.com/ukscone/set_overscan/archive/master.zip```
3. ```$ unzip master.zip && rm master.zip```
4. ```$ cd set_overscan-master```
5. ```$ sudo make```
6. ```$ sudo ./set_overscan.sh```

Or edit ```/boot/config.txt``` manually.

```
overscan_left=24
overscan_right=24
overscan_top=24
overscan_bottom=24
```

#### Audio

```
$ amixer sset PCM,0 100%
$ speaker-test -c2 -t wav
```

#### RetroFlag

If using a RetroFlag case, this will set up the required script.

1. ```$ wget -O - "https://raw.githubusercontent.com/RetroFlag/retroflag-picase/master/install.sh" | sudo bash```
2. ``` $ sudo nano /opt/RetroFlag/SafeShutdown.py```

Edit lines 19 and 23 to exclude references to emulation and sleep.

```
# line 19
# os.system("sudo killall emulationstation && sleep 5s && sudo shutdown -h now")
os.system("sudo shutdown -h now")

# line 23
# os.system("sudo killall emulationstation && sleep 5s && sudo reboot")
os.system("sudo reboot")
```

## SDL 2.0.8

> SDL2 is now installed by default but it is compiled for desktop users with support for X and OpenGL, neither of which we can take advantage of in a framebuffer and it will just confuse MAME if you try to compile using it.

> Any program that uses SDL2 to draw to the framebuffer should now be doing it in hardware on the raspberry pis videocore4 using opengles2!

#### Uninstall SDL

```libsdl2-dev is``` not installed on raspbian stretch lite. If using the desktop version, you must uninstall it.

1. ```$ sudo apt-get remove -y --force-yes libsdl2-dev```
2. ```$ sudo apt-get autoremove -y```

#### Add Dependencies

Add packages for build environment.

```
$ sudo apt-get install libfontconfig-dev qt5-default automake mercurial libtool libfreeimage-dev libopenal-dev libpango1.0-dev libsndfile-dev libudev-dev libtiff5-dev libwebp-dev libasound2-dev libaudio-dev libxrandr-dev libxcursor-dev libxi-dev libxinerama-dev libxss-dev libesd0-dev freeglut3-dev libmodplug-dev libsmpeg-dev libjpeg-dev
```

#### SDL

Create development directory.

1. ```cd ~```
2. ```mkdir development```
3. ```cd development```

Get the latest libsdl build.

``` $ hg clone http://hg.libsdl.org/SDL ```

Build libsdl.

1. ```$ cd SDL```
2. ```./autogen.sh```
3. ```./configure --disable-pulseaudio --disable-esd --disable-video-mir --disable-video-wayland --disable-video-opengl --host=arm-raspberry-linux-gnueabihf```
4. ```$ make```
5. ```$ sudo make install```
6. ```$ cd ../```

#### SDL Libraries

Download all the utility libraries.

1. ```$ cd ~/development```
2. ```$ wget http://www.libsdl.org/projects/SDL_image/release/SDL2_image-2.0.2.tar.gz```
3. ```$ wget http://www.libsdl.org/projects/SDL_mixer/release/SDL2_mixer-2.0.2.tar.gz```
4. ```$ wget http://www.libsdl.org/projects/SDL_net/release/SDL2_net-2.0.1.tar.gz```
5. ```$ wget http://www.libsdl.org/projects/SDL_ttf/release/SDL2_ttf-2.0.14.tar.gz```

Uncompress all the utility libraries.

1. ```$ tar zxvf SDL2_image-2.0.2.tar.gz```
2. ```$ tar zxvf SDL2_mixer-2.0.2.tar.gz```
3. ```$ tar zxvf SDL2_net-2.0.1.tar.gz```
4. ```$ tar zxvf SDL2_ttf-2.0.14.tar.gz```

Build the Image file loading library.

1. ```$ cd SDL2_image-2.0.2```
2. ```$ ./autogen.sh```
3. ```$ ./configure```
4. ```$ make```
5. ```$ sudo make install```
6. ```$ cd ../```

Build the Audio mixer library.

1. ```$ cd SDL2_mixer-2.0.2```
2. ```$ ./autogen.sh```
3. ```$ ./configure```
4. ```$ make```
5. ```$ sudo make install```
6. ```$ cd ../```

Build the Networking library.

1. ```$ cd SDL2_net-2.0.1```
2. ```$ ./autogen.sh```
3. ```$ ./configure```
4. ```$ make```
5. ```$ sudo make install```
6. ```$ cd ../```

Build the Truetype font library.

1. ```$ cd SDL2_ttf-2.0.14```
2. ```$ ./autogen.sh```
3. ```$ ./configure```
4. ```$ make```
5. ```$ sudo make install```
6. ```$ cd ../```

#### Done

Optionally, you can remove your development folder if not compiling or installing anything else.

```$ rm -rf ~/development```

## MAME

#### Compile

1. ```$ sudo nano /etc/dphys-swapfile```
2. Change ```CONF_SWAPSIZE=100``` to ```CONF_SWAPSIZE=2048```.
3. Save and reboot.
4. ```$ cd development```
5. ```$ wget -O - https://github.com/mamedev/mame/releases/download/mame0203/mame0203s.zip > mame0203s.zip && unzip mame0203s.zip - d ~/```
6. ```$ cd ~/```
7. ```$ make -j5```
8. ```$ sudo nano /etc/dphys-swapfile```
9. Change ```CONF_SWAPSIZE=2048``` back to ```CONF_SWAPSIZE=100```.
10. Save and exit.

#### Precompiled

Precompiled mame at [choccyhobnob](https://choccyhobnob.com/mame/mame/). Check and substitute address accordingly for the latest package. You can use wget directly on the pi, or download on your mac and use scp to copy to your pi.

1. ```$ cd development```
2. ```$ wget -O - https://choccyhobnob.com/?d=1964 > mame0203b_rPi.zip && unzip mame0203b_rPi.zip -d ~/```
3. ```$ cd ~/```
4. ```$ mv mame0203b_rpi mame0203b```

#### Run

To generate config, run ```$ ./mame -cc```.

To run a game, run ```$ ./mame romname```.

## AdvancedMAME

1. ```$ cd development```
2. ```$ wget https://github.com/amadvance/advancemame/releases/download/v3.9/advancemame_3.9-1_armhf.deb```
3. ```$ sudo apt install ~/development/advancemame_3.9-1_armhf.deb```

You can now run the command ```$ advmame``` to create configuration files, placed in the ```~/.advance``` directory. Binary is located in ```/usr/local/bin```.

To run a game, run ```$ advmame romname```.

## Attractmode

#### SFML

1. ```$ cd ~/development```
2. ```$ wget -O - https://github.com/mickelson/sfml-pi/archive/master.zip > sfml-pi-master.zip && unzip sfml-pi-master.zip```
3. ```$ cd sfml-pi-master/cmake```
4. ```$ cmake .. -DSFML_RPI=1 -DEGL_INCLUDE_DIR=/opt/vc/include -DEGL_LIBRARY=/opt/vc/lib/libbrcmEGL.so -DGLES_INCLUDE_DIR=/opt/vc/include -DGLES_LIBRARY=/opt/vc/lib/libbrcmGLESv2.so```
5. ```$ sudo ldconfig```

#### FFmpeg

1. ```$ cd ~/development```
2. ```$ wget https://ffmpeg.org/releases/ffmpeg-4.1.tar.bz2```
3. ```$ tar -xf ffmpeg-4.1.tar.bz2```
4. ```$ cd ffmpeg-4.1```
5. ```$ ./configure --enable-mmal --disable-debug --enable-shared```
6. ```$ make```
7. ```$ sudo make install```
8. ```$ sudo ldconfig```

#### AttractMode

1. ```$ cd ~/development```
2. ```$ wget -O - https://github.com/mickelson/attract/archive/master.zip > attract-master.zip && unzip attract-master.zip```
3. ```$ cd attract-master```
4. ```$ make USE_GLES=1```
5. ```$ sudo make install USE_GLES=1```

To run attractmode, run ```$ attract```. Binary is located in ```/usr/local/bin```.

## Load on Login

As suggested under config settings part of the guide, be sure you used ```$ sudo raspi-config``` to set the following:

* *3 Boot Options* -> *B1 Desktop / CLI* -> *B2 Console Autologin*

```.bash_profile``` of the user is executed to configure your shell before the initial command prompt. Our script will launch attract if the current user (pi) logs in and it is not an ssh connection.

1. ```$ cd ~```
2. ```$ touch .bash_profile```
3. ```$ chmod -x .bash_profile```
4. ```$ nano .bash_profile```

```
#!/bin/sh
# if SSH connection do nothing, else launch emulationstation
# (first construct checks that SSH_CONNECTION is set to something)
[ -n "${SSH_CONNECTION}" ] || /usr/local/bin/attract
```

You may replace the path to the attract binary with advmame (single rom instance?), or anything else you wish to have automatically loaded.

## Bluetooth

I am using an 8bitdo m30 controller. However, this should work similar for most other controllers.

1. Enter bluetoothctl to open Bluetooth control.
2. Put the remote into d-input mode by holding start + B until LED 1 blinks.
3. Put the remote into pairable mode by holding the small pair button on the front of the remote for 2 seconds. LEDs will sequence.
4. At the [bluetooth]# prompt enter the following commands:

```
[bluetooth]# discoverable on
[bluetooth]# pairable on
[bluetooth]# agent on
[bluetooth]# default-agent
[bluetooth]# scan on
Discovery started
[CHG] Controller B8:27:EB:7B:89:3C Discovering: yes
[NEW] Device E4:17:D8:87:09:7D 8BitDo M30 gamepad
[bluetooth]# pair E4:17:D8:87:09:7D
[bluetooth]# trust E4:17:D8:87:09:7D
[bluetooth]# scan off
[bluetooth]# paired-devices
Device E4:17:D8:87:09:7D 8BitDo M30 gamepad
[bluetooth]#quit
```

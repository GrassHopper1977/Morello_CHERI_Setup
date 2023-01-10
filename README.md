# Morello CHERI Setup
My notes on setting up the Morello CHERI board with CheriBSD, upgrading it and various things I've noticed.

# Initial Setup and Install
## Things You Need to Know!
### General Setup
- Some instructions come with the Morello Cheri board. They are pretty confusing and will make your life difficult. For those of us on the DSbD program, we are suopposed to be trying to create things with the Morello Cheri rather than actively contributing to the development of it and building the OS, etc. We are better off using one of teh pre-built OS images to start with.
- There are no SPI, I2C or serial ports. If you were hoping to do some clever embedded interfacing then you'd better hope that you can do it all via USB. Thi will be easier if you use Android or Linux - FreeBSD and, by extension, CheriBSD aren't so well supported on the drivers for periphiperals from. We were trying to find a USB to CAN device that was supported by FreeBSD and ended up having to use [libusb](https://libusb.info/) (which had it's own problems).
### Operating Systems
- There are three pre-built OS available:
1. CheriBSD - A CHERIfied version of FreeBSD. This is the most developed, most complete and the only one I've played with.
2. Android - I haven't tried this
3. Linux - I haven't tried this either.
- I used the guide [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/cover/index.html) and added my own notes as I went along to create this.
### CheriBSD
- [CheriBSD](https://www.cheribsd.org/) is a "cherified" version of [FreeBSD](https://www.freebsd.org/)
- CheriBSD can be run in four different ways. There are two builds:
1. Hybrid
2. Pure-Capabilities
- Addtionally each build has two kernels:
1. Hybrid
2. Pure-Capabilities
- Using the Hybrid versions of both is the most likely to work but doesn't give you much in the way of additional protection.
- I used the Pure-Capabilities ABI with the Hybrid Kernel (the dfeault) not realising that I wasn't using the Pure-Caps kernel. You can choose a different kernel from the FreeBSD boot menu, but we'll go through that later.

## Connecting to the Morello Board
The Morello board has a USB type B connectopr on it. Connect from that to a PC runnnign WIndows or Linux (I use Ubuntu with a GUI) and you will find that it adds 4 serial ports and the USB drive.

### The USB Drive
The USB drive is an onboard Sd card on teh Morello. It is used to upgrade teh frimware on teh Morello board (which you should always do before installing an OS). The drive is called `M1SDP`.

### The Serial Ports
The actual names of teh serial ports will depend upon your OS. On Ubuntu they are called `/dev/ttyUSB0` to `/dev/ttyUSB3`. On *NIX systems like Linux and FreeBSD you may need to add your user to teh `dialer` group in order to access teh serial ports. The serial ports are all run at `115200` buad, 8 data bits, 1 stop bit, No parity and XON/XOFF flowcontrol.

#### Serial Port 0 - MCC
The is the MCC Morello Management Controller. We can use this to set teh clock, reboot independant of teh OS and it is where we can see the firmware updates fro teh Morello board being applied.

### Serial Port 1 - PCC
I have no idea what this is used for. Some form of debugging.

### Serial Port 2 - Morello System Console
This is where we will be able to access our CheriBSD installation's console. remember that some of teh CheriBSD builds don't have a driver for teh HDMI yet (fixed from V22.12 but there are reports of isseus still).

### Serial port 3 - ?
I've never used it.

# CheriBSD Notes
1. If you have a USB hub plugged when booting it causes an error that looks like this:
```
    _____ _               _ ____   _____ _____
   / ____| |             (_)  _ \ / ____|  __ \
  | |    | |__   ___ _ __ _| |_) | (___ | |  | |
  | |    | '_ \ / _ \ '__| |  _ < \___ \| |  | |
  | |____| | | |  __/ |  | | |_) |____) | |__| |
   \_____|_| |_|\___|_|  |_|____/|_____/|_____/
                                                 ```                        `
                                                s` `.....---.......--.```   -/
 /---------- Welcome to CheriBSD ----------\    +o   .--`         /y:`      +.
 |                                         |     yo`:.            :o      `+-
 |  1. Boot Multi user [Enter]             |      y/               -/`   -o/
 |  2. Boot Single user                    |     .-                  ::/sy+:.
 |  3. Escape to loader prompt             |     /                     `--  /
 |  4. Reboot                              |    `:                          :`
 |  5. Cons: Serial                        |    `:                          :`
 |                                         |     /                          /
 |  Options:                               |     .-                        -.
 |  6. Kernel: default/kernel (1 of 2)     |      --                      -.
 |  7. Boot Options                        |       `:`                  `:`
 |                                         |         .--             `--.
 |                                         |            .---.....----.
 \-----------------------------------------/
   Autoboot in 0 seconds. [Space] to pause

Loading kernel...
/boot/kernel/kernel text=0x2e0 text=0x858890 text=0x24a9b8 text=0x30 data=0x2631
e0 data=0x0+0x357d50 0x8+0x11ba28+0x8+0xe9165
Loading configured modules...
/etc/hostid size=0x25
/boot/entropy size=0x1000
No valid device tree blob found!
WARNING! Trying to fire up the kernel, but no device tree blob found!
EFI framebuffer information:
addr, size     0xfe000000, 0x7e9000
dimensions     1920 x 1080
stride         1920
masks          0x00ff0000, 0x0000ff00, 0x000000ff, 0xff000000
```
To work around this, unplug the USB devcies and reboot (you can execute a `reboot` from the MCC console).

# Upgrading from V22.05 to V22.12
First we need to update the firmware on the Morello board.


## Upgrade the Morello Firmware
We follow the basic instructions from here [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/morello-firmware/index.html) but with a little more detail.
1. Use the MCC (port 0 - the first serial port) to double check that the clock is correct (mine had drifted badly).

```
Cmd> debug
Debug> time
03 : 41 : 55

Change Time? Y\N >y
s:>45
m:>8
h:>16
Debug> date
08/01/2023

Change Date? Y\N >y
D:>9
M:>1
Y:>2023
Debug> exit
```

2. Download the [prebuilt firmware images from ARM via the Morello GitLab instance](https://git.morello-project.org/morello/board-firmware). Download the whole directory as a zip (or tar, if you prefer).
3. Unzip the files (Windows can do this by right clicking on the archive, on Ubuntu you can use the Archive Manager).
4. Now we're going to copy teh files into place on the Merllo's internal 2GB card. I like to ensure that CheriBSD is shutdown safely before I mess with these things. If your CheriBSD is still runnning then connect to it (port 2 - the third serial port) and tell it to shutdown safely.
```
shutdown -p now
```
You may notice the device powering down messages on MCC (port 0), if you are looking.
5. Now we need to copy the files that we extracted into the internal card on the Morello. On Ubuntu the drive should automatically mount as `M1SDP`. If it doesn't automatically appear you may need to turn on the USB using the MCC (port 0) command `USB_ON` (it should still be on though).
6. Delete the existing file and replace them with the new files (you can use the shell but I prefer the UI on Ubuntu). There will be sevral folders and files to copy into the root of the flash device. They are:
```
LIB
LICENSES
MB
SOFTWARE
config.txt
ee0364b.txt
```
7. We need to unmount the USB to ensure that it has finished syncing before we reboot (You could execute `sync` from the shell on the device that you're copying from. It is still suggested to unmount the device even after a sync). Unbuntu's Files has an unmount option for these drives so use that.
8. Now we return to the MCC (serial 0) and we reboot the Morello:
```
Cmd> reboot

Powering up system...

MCC to access SD card request acknowledged.
Clear ULINK JTAG

Time :  16:59:58
Date :  09:01:2023

Switching on ATXPSU...
Switching on main power...

12V Rail Check Pass

Reading Board File \MB\HBI0364B\io_v010f.txt
TOTAL OSCCLKS: 15
PMIC RAM configuration (80_1_86G.bin)...

Temperature (deg C)
Outlet = 25.4 PMIC   = 27.9 SoC = 27.4 IN2  = 26.9 IN1 = 27.9
MidBrd = 27   PCIeSW = 24   MCC = 30.3 FPGA = 27.9 SoC Fan Speed = 9%
ALERT off

Configuring motherboard (rev B, var A)...

Configuring FPGA from file \MB\HBI0364B\io_v010f.bit
Address: 0x00240000
```
It will reboot several times and will update. you can watch it happening from the MCC (serial 0).

## Updating CheriBSD
We follow the basic instructions from [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/getting/index.html) but with a little more detail.
1. We need to download the memstick image of the latest CheriBSD build from [here](https://www.cheribsd.org/)
2. IMPORTANT! If you haven't yet updated the firmware of the Morello board go back and do that now.
3. Write the disk image that you've downloaded onto an SD card. Ubuntu has a UI for this if you just click the file. Otherwise you can do it from the console by executing something like this (change DISK to teh name of your USB stick):
```
dd if=cheribsd-memstick-arm64-aarch64c-22.12.img of=/dev/DISK bs=1048576
```
4. 

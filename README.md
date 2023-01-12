# Morello CHERI Setup
My notes on setting up the Morello CHERI board with CheriBSD, upgrading it and various things I've noticed.

# Initial Setup and Install
## Things You Need to Know!
### General Setup
- Some instructions come with the Morello Cheri board. They are pretty confusing and will make your life difficult. For those of us on the DSbD program, we are suopposed to be trying to create things with the Morello Cheri rather than actively contributing to the development of it and building the OS, etc. We are better off using one of teh pre-built OS images to start with.
- There are no SPI, I2C or serial ports. If you were hoping to do some clever embedded interfacing then you'd better hope that you can do it all via USB. Thi will be easier if you use Android or Linux - FreeBSD and, by extension, CheriBSD aren't so well supported on the drivers for periphiperals from. We were trying to find a USB to CAN device that was supported by FreeBSD and ended up having to use [libusb](https://libusb.info/) (which had it's own problems).
- Until V22.12 there was no desktop environment. There's not a lot of notes on using the desktop because of this fact.
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
This is where we will be able to access our CheriBSD installation's console. Remember that some of teh CheriBSD builds don't have a driver for teh HDMI yet (fixed from V22.12 but there are reports of isseus still).

### Serial port 3 - ?
I've never used it.

# CheriBSD Notes
## Gotchas - Purecaps
### I've Installed the Purecaps Build so Everyhting is Secured
No, it's not. If you use teh default kernel on the Pure-Caps build then you are still using the Hybris Kernel. You will need to change the kernel during the boot.
### I'm Using Purecaps So I Can't Compile or Run Hybrid Code
No, that's not true. Both builds can run with eitehr kernel and can run code built for either kernel.
### Some of My Code Doesn't Work When Built for Purecaps but Works on Hybrid so My Code Is Wrong!
Maybe not - this can happen for a number of reasons. Often, it is tempting to blame our code. I like to try my code out on a separate FreeBSD machine first and then I test it on Pure-Caps and Hybrid.
### I Can Compare Library Function Calls on hybrid and Pure-Cap Versions
Probably not. When the OS is built they build everything twice. There is a purecaps version and a hybrid version of teh whole library. If a library function isn't working on purecaps, you can't just compare what you're passing in each case as the memory addresses will be completely different and any structs will by aligned differently for purecaps. In general, if you'ev found something that work in Hybrid but not Purecaps then you have probably found a bug. Check on teh Slack channel first and then report it on GitHub.

## How Do I use the Full-Caps Kernel?

## CheriBSD Won't Boot!
If you have a USB hub plugged when booting it causes an error that looks like this:
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
To perform an upgrade I just went through a full installation again. I updated the notes on Installing CheriBSD to reflect the V22.12.

# Installing CheriBSD
First we need to update the firmware on the Morello board.

## Upgrade the Morello Firmware
We follow the basic instructions from here [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/morello-firmware/index.html) but with my own notes. The instruction on upgrading teh firmware are a little unclear so I've clarified a few points here.
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

## Installing CheriBSD
We follow the basic instructions from [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/getting/index.html) but with my own notes. The CheriBSD install instrcutions are much easier to follow than the ARM firmware instructions.
1. We need to download the memstick image of the latest CheriBSD build from [here](https://www.cheribsd.org/)
2. IMPORTANT! If you haven't yet updated the firmware of the Morello board go back and do that now.
3. Write the disk image that you've downloaded onto an SD card. Ubuntu has a UI for this if you just click the file. Otherwise you can do it from the console by executing something like this (change DISK to the name of your USB stick):
```
dd if=cheribsd-memstick-arm64-aarch64c-22.12.img of=/dev/DISK bs=1048576
```
4. Now put the USB stick into the Morello (unplug any other USB devices) and reboot the machine (use the MMC and enter `reboot`).
5. After a short time the Morello console (serial port 2) will display
```
Press ESCAPE for boot options .....
```
6. At this point you should press escape and teh Boot Manager will load:
```

 Morello System Development Platform
 Morello-R0P1                                        2.50 GHz
 EDK II                                              16352 MB RAM



   Select Language            <Standard English>         This is the option
                                                         one adjusts to change
 > Device Manager                                        the language for the
 > Boot Manager                                          current system
 > Boot Maintenance Manager

   Continue
   Reset







  ^v=Move Highlight       <Enter>=Select Entry
```
7. Use the arrow keys to select `Boot Manager` then press Enter.
8. Use the arrow keys to select your USB stick then press Enter
9. Now we get teh CheriBSD splash screen. Yiou can wait for it to count down or just press Enter to skip the countdown. Yo may also get teh same output on a screen connected to the HDMI - it's best to stick to the serial port as this is well documented as working.
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
 |  1. Boot Installer [Enter]              |      y/               -/`   -o/
 |  2. Boot Single user                    |     .-                  ::/sy+:.
 |  3. Escape to loader prompt             |     /                     `--  /
 |  4. Reboot                              |    `:                          :`
 |  5. Cons: Serial                        |    `:                          :`
 |                                         |     /                          /
 |  Options:                               |     .-                        -.
 |  6. Kernel: default/kernel (1 of 1)     |      --                      -.
 |  7. Boot Options                        |       `:`                  `:`
 |                                         |         .--             `--.
 |                                         |            .---.....----.
 \-----------------------------------------/
   Autoboot in 8 seconds. [Space] to pause
```
10. Various boot log messages will appear.
11. You will be asked to choose your console type:
```
Welcome to FreeBSD!

Please choose the appropriate terminal type for your system.
Common console types are:
   ansi     Standard ANSI terminal
   vt100    VT100 or compatible terminal
   xterm    xterm terminal emulator (or compatible)

Console type [xterm]:
```
12. It is safe to just hit enter and accept `xterm`.
13. You're now into the FreeBSD Installer.
14. At this screen press Enter:
```

                                                          lqqqqqqqqqqqquWelcometqqqqqqqqqqqk
                                                          x Welcome to FreeBSD! Would you  x
                                                          x like to begin an installation  x
                                                          x or use the live CD?            x
                                                          tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                                          x  [Install] [ Shell ] [Live CD] x
                                                          mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj

```
15. Choose the required keyboard type from the list using the arrow keys and Enter.
16. The screen will jump back to teh top of teh menu and offer you teh chance to test the keyboard map. You can test it or just select the option above and choose `Continue with uk.kbd keymap`.
17. You are asked to name your machine. Give it a suitable name and press Enter.
18. In the Partitioning options select `Auto (UFS)` and press enter
19. Select `Entire Disk` and press Enter.
20. Select `Yes` to confirm that you are erasing the whole disk.
21. It will show you the proposed partition scheme. Just select `Finish` and press Enter.
22. Select `Commit` and press Enter to continue.
23. Now a long installation process happens. Lots of waiting...
24. If you are upgrading you may receive the following message:
```
lqqqqqBoot Configurationqqqqqqqqk
x There are multiple "FreeBSD"  x
x EFI boot entries. Would you   x
x like to remove them all and   x
x add a new one?                x
tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
x     < Yes >   < No >          x
mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
```
Just select Yes and press Enter
25. You will be asked to create the root password. Choose a suitable password and save it in a password manager so you don't lose it.
26. You will be asked if you want to configur ethe Ethernet networking. I would suggest that you do. Select `OK` and press Enter.
27. Would you like to configure IPv4 for this interface? Select `Yes` and press Enter.
28. Would you like to use DHCP to configure this interface? Select `Yes` and press Enter.
29. Would you like to configure IPv6 for this interface? Select `Yes` and press Enter.
30. Would you like to try stateless address autoconfiguration (SLAAC)? Select `Yes` and press Enter.
31. There will be a box with Resolver Configuration in it.
```
                                        lqqqqqqqqqqqqqqqqqqqqqqquNetwork Configurationtqqqqqqqqqqqqqqqqqqqqqqqk
                                        x Resolver Configuration                                              x
                                        x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
                                        x xSearch         home                                              x x
                                        x xIPv6 DNS #1    fe80::d686:60ff:fe99:5371%re0                     x x
                                        x xIPv6 DNS #2                                                      x x
                                        x xIPv4 DNS #1    192.168.1.254                                     x x
                                        x xIPv4 DNS #2                                                      x x
                                        x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
                                        tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                        x                        [  OK  ]     [Cancel]                        x
                                        mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj

```
32. Select `OK` and press Enter.
33. is teh system clock set to UTC? Select `Yes` and press Enter.
34. Select a region. I'm in Europe so I select that with the arrow keys and press Enter.
35. Now we're asked to choose a country. I use teh arrow keys to select `United Kingdom of Great Britain and Northern Ireland` and press Enter.
36. `Does teh abbreviation 'GMT look reasonable?` Yes, that is my timezone so I select `Yes` and press Enter.
37. Double check the date is correct. Mine is so I select `Skip` and press Enter.
38. Double check the time is correct. Mine is so I select `Skip` and press Enter.
39. We're offered this screen:
```

                                    lqqqqqqqqqqqqqqqqqqqqqqqqqqquSystem Configurationtqqqqqqqqqqqqqqqqqqqqqqqqqqqk
                                    x Choose the services you would like to be started at boot:                  x
                                    x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
                                    x x[ ] local_unbound      Local caching validating resolver                x x
                                    x x[X] sshd               Secure shell daemon                              x x
                                    x x[ ] moused             PS/2 mouse pointer on console                    x x
                                    x x[ ] ntpd               Synchronize system and network time              x x
                                    x x[ ] ntpd_sync_on_start Sync time on ntpd startup, even if offset is highx x
                                    x x[ ] powerd             Adjust CPU frequency dynamically if supported    x x
                                    x x[X] dumpdev            Enable kernel crash dumps to /var/crash          x x
                                    x x[ ] debugger_on_panic  Run debugger on kernel panic                     x x
                                    x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
                                    tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                    x                                  [  OK  ]                                  x
                                    mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
```
40. We can select any additional services that we would like using the arrow keys and pressing the space bar. I have selected `ntp`, `nptd_sync_on_start` and `debugger_on_panic`. Then press Enter to accept the choices.
41. Add User Accounts This is important if you're going to use the CHERI Desktop Environment. I'm going to install it so I create 1 additional user, accepting all the defaults and setting a password. You cannot log into the Desktop environment as the root user so you should add another user here.
42. New for V22.12! You must be online to install this and it could take a long time to finish.
```
                                                          lqqqqqqqqquCHERI Desktoptqqqqqqqqk
                                                          x Would you like to install the  x
                                                          x CHERI desktop environment      x
                                                          x (requires network)?            x
                                                          tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                                          x      [ Yes  ]     [  No  ]     x
                                                          mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
```
43. I select `Yes` and press Enter. This is the perfect time to go and get a drink as it will take a little while to finish...
44. If you're installing the CHERI Desktop Environment you won't be able to log in as root. You will need to set your additonal users to be members of the `video` group. You will be asked to select which user you want to add to teh group. Use arrow keys and space to select the users and then and press enter.
```

                                                      lqqqqqqqqqqqqquCHERI Desktoptqqqqqqqqqqqqqk
                                                      x Users must be in the video group to log x
                                                      x in to a desktop environment. Choose any x
                                                      x additional users to add to the group:   x
                                                      x lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk x
                                                      x x            [ ] Alan Alan            x x
                                                      x mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj x
                                                      tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                                      x                [  OK  ]                 x
                                                      mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
```
46. Final Configuration - You're offered the chance to change anything you've done. You can use the arrow keys and space to select from the menu. If you have nothing to change then just press Enter.
47. Manual Configuration - Do we want to drop to the shell and make any extra changes. We don't so select `No` to press Enter.
48. Complete. Note: DO NOT remove the USB stick just yet (I did that on a FreeBSD install once and it trashed everything!)
```
                                                         lqqqqqqqqqqqquCompletetqqqqqqqqqqqqqk
                                                         x Installation of FreeBSD complete! x
                                                         x Would you like to reboot into the x
                                                         x installed system now?             x
                                                         tqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqu
                                                         x [ Reboot ] [Shutdown] [Live CD ]  x
                                                         mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
```
Select `Reboot` and press Enter.
49. Now that you've pressed Enter you can safely remove the USB stick. It is possible that it won't reboot on it's own. Wait until you get teh message about your uptime - after that you can safely force the reboot from the MCC (serial port 0) by typing `reboot`.
50. The boot menu will appear. Either wait for the timeout or press enter to boot in the default (Hybrid) kernel.
```    _____ _               _ ____   _____ _____
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
 |  6. Kernel: default/kernel (1 of 4)     |      --                      -.
 |  7. Boot Options                        |       `:`                  `:`
 |                                         |         .--             `--.
 |                                         |            .---.....----.
 \-----------------------------------------/

```
51. It will go through the boot up procedure and then give you a login prompt. Now you can log in as one of teh users that you created or as the root user.
```
Thu Jan 12 08:54:03 GM
CheriBSD/arm64 (cheribsd.local) (ttyu0)

login: root
Password:
Jan 12 08:55:57 cheribsd login[926]: ROOT LOGIN (root) ON ttyu0
FreeBSD 14.0-CURRENT #0 8993e1c3bba: Thu Dec 15 23:25:54 UTC 2022     jenkins@ctsrd-build-linux-xx:/local/scratch/jenkins/workspace/CheriBSD-pipeline_releng_22.12/cheribsd-morello-purecap-build/local/scratch/jenkins/workspace/CheriBSD-pipeline_releng_22.12/cheribsd/arm64.aarch64c/sys/GENERIC-MORELLO

Welcome to CheriBSD!

CheriBSD extends FreeBSD to implement memory protection and software
compartmentalization features enabled by CHERI-extended CPUs.

The CheriBSD front page:
  https://www.cheribsd.org/

We provide support via a mailing list:
  https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheri-lists.html

And also via the CHERI-CPU Slack:
  https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheri-slack.html

CheriBSD source may be found at:
  https://github.com/CTSRD-CHERI/cheribsd/

Find out more about about CHERI at https://cheri-cpu.org/

WARNING: INVARIANTS kernel option defined, expect reduced performance
WARNING: WITNESS kernel option defined, expect reduced performance
root@cheribsd:~ #
```

=== Setting Up the Development Environment
So you've installed CheriBSD but you want to write some code and commit changes to a repository so we're going to need to install the compiler and git and go through setting them all up.

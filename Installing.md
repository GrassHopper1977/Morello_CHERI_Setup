# Installing Morello CheriBSD
First we need to update the firmware on the Morello board and then we will install CheriBSD.

# Upgrade the Morello Firmware
We follow the basic instructions from here [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/morello-firmware/index.html) but with my own notes. The instruction on upgrading the firmware are a little unclear so I've clarified a few points here.
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
4. Now we're going to copy the files into place on the Merllo's internal 2GB card. I like to ensure that CheriBSD is shutdown safely before I mess with these things. If your CheriBSD is still runnning then connect to it (port 2 - the third serial port) and tell it to shutdown safely.
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

# Installing CheriBSD
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

# Preparing the Dev Environment
Note: All of these instructions are entered into the CheriBSD console (serial port 2).

## Notes on Installing Packages
There's a detaailed description of the packages and how they work [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/packages/index.html) but here's a simplified version.
Normally you would install 3rd party packages using `pkg`. CheriBSD doesn't have `pkg` yet instead it has two different package managers:
1. pkg64 - This is used to install Hybrid ABI packages. This are almost identical to the FreeBSD packages. Tehy do not have the improvements in memory protection or software compartmentalisation.
2. pkg64c - This are teh Cheri ABI packages. This have been built with the extra protections that Cheri provides. They may have errors and are consodered experimental. There's till a lot of packages to be converted to Cheri though.

## So Which Package Manager Do I Use?
### CheriABI (`pkg64c`)
If the package is being used for my program then I try to use the CheriABI (so use `pkg64c`) packages first. If they don't exist or I have issues then I switch to the Hybrid BI version (using `pkg64` instead). Almost everything I'm doign is experimental anyway, so this would seem to make sense.
### Hybrid ABI (`pkg64`)
If the package is just a background task or it is critical that it works (e.g. `nano` or `git`) then I'll use the Hybrid version as it is more likely to be safe.

## Installing Packages That You Need
So you've installed CheriBSD but you want to write some code and commit changes to a repository so we're going to need to install the compiler and git and go through setting them all up.

### 1. Install GIT
1. Enter this `pkg64 install git`
2. If you have some code on GitHub to check out then, you may wish to save your GitHub credentials locally on the machine (my machine is behind a firewall and therefor reasonably safe). The best way to do this is to create a personal access token to use instead of a password.
3. Instructions for creating a personal access token can be found [here](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
4. From your account (or root) home directory:
```
mkdir github
cd github
git clone <PATH TO REPO>
```
5. When prompted enter your username and use your personal Access Token for the password.
### 2. Install nano
`pkg64 install nano`
### 3. Install Compiler & Debugger
`pkg64 install llvm-base`
### 4. (Optional) Install wget
`pkg64 install wget`

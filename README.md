# Morello CHERI Setup with FreeBSD
My notes on setting up the Morello CHERI board with CheriBSD, upgrading it, using it communicate with actual hardware, and various things I've noticed.

# CheriBSD and Morello Hardware
If you are planning on integrating with external devices you should note:
- There are no physical serial ports (although USB to serial works)
- There are no I2C ports available (there is one on the HDMI interface but I was unable to access that in V22.05)
- There are no SPI ports available. There are SPI on the board but they are all in use on the board.
- If you're goin to be interfacing to the other devices it will have to be via ethernet or USB.
- FreeBSD and, by extension, CheriBSD doesn't have as many drivers as Linux.
- If you are trying to talk to a USB device that doesn't already have a driver then you can either write you have three choices:
  1. Create your own hardware (or find some pre-existing hardware) that can convert serial to whatever protocol you need.
  2. Write you own USB driver. This doesn't look as difficult as you might think. There is a guide [here](https://freebsdfoundation.org/wp-content/uploads/2021/11/Simple_USB_Driver_for_FreeBSD.pdf). This guide also discusses how to sniff USB packets to aid in writing a driver.
  3. Use [libusb](https://libusb.info/) - this library is pretty standardised so the same code shoudl function in the same way on Linux, FreeBSD, CheriBSD & Windows.
- There is a USB bug in V22.05 & V22.12 that prevents USB Bulk data from working but there is now a custom kernel to fix it (you just have to ask on the CheriBSD Slack channel). If you are interested the ticket for this issue can be found [here](https://github.com/CTSRD-CHERI/cheribsd/issues/1616).
- There are drivers for some USB to I2C devices though I have yet to try them. For example [CP2112](https://www.freebsd.org/cgi/man.cgi?query=cp2112&sektion=4) is supported.

# Installing an Operating System
## Before You Start
- Some instructions come with the Morello Cheri board. They are pretty confusing and will make your life difficult. For a quick start you are better off installing one of the pre-built OS images than trying to build one from source code (there will be time for that later).
- There are three pre-built OS available:
  1. CheriBSD - A CHERIfied version of FreeBSD. This is the most developed, most complete and the only one I've played with.
  2. Android - I haven't tried this.
  3. Linux - I haven't tried this either.

## Connecting to the Morello Board
The Morello board has a USB type B connector on it. Connect from that to a PC runnning a suitable OS (I use Ubuntu with a GUI) and you will find that it adds 4 serial ports and the USB drive.

### The USB Drive
The USB drive is the Morello's onboard SD card. It is used to upgrade the frimware on the Morello board (which you should always do before installing an OS). The drive is called `M1SDP`.

### The Serial Ports
The actual names of the serial ports will depend upon your OS. On Ubuntu they are called `/dev/ttyUSB0` to `/dev/ttyUSB3`. On *NIX systems like Linux and FreeBSD you may need to add your user to the `dialer` group in order to access these serial ports. The serial ports are all run at `115200` buad, 8 data bits, 1 stop bit, No parity and XON/XOFF flow control.
each port provide different functionality.

#### Serial Port 0 - MCC
The is the MCC Morello Management Controller. We can use this to set the clock, reboot 
(independantly of the OS) and it is where we can see the firmware updates for the Morello board being applied.

### Serial Port 1 - PCC
I have no idea what this is used for. It look slike debugging inforamtion.

### Serial Port 2 - Morello System Console
This is where we will be able to access our CheriBSD installation's console. Remember that some of the CheriBSD builds don't have a driver for the HDMI yet (fixed from V22.12 but there are reports of issues still).

### Serial port 3 - ?
I've never used it. It looks like more debugging information.

## Notes on CheriBSD
- Until V22.12 there was no desktop environment. There's not a lot of notes on using the desktop because of this fact.
- [CheriBSD](https://www.cheribsd.org/) is a "cherified" version of [FreeBSD](https://www.freebsd.org/)
- CheriBSD can be run in four different ways. There are two builds:
  1. Hybrid
  2. Pure-Capabilities
- Addtionally each build has two kernels:
  1. Hybrid
  2. Pure-Capabilities
- Using the Hybrid versions of both is the most likely to work but doesn't give you much in the way of additional protection.
- I used the Pure-Capabilities ABI with the Hybrid Kernel (the dfeault) not realising that I wasn't using the Pure-Caps kernel. You can choose a different kernel from the FreeBSD boot menu, but we'll go through that later.

## Installing CheriBSD
These are my [full installation instructions for CheriBSD V22.12](installing.md).



## Known Issue with Morello
A list in maintained [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/morello-issues/index.html).

# CheriBSD Notes
## Gotchas - Purecaps
### I've Installed the Purecaps Build so Everyhting is Secured
No, it's not. If you use the default kernel on the Pure-Caps build then you are still using the Hybris Kernel. You will need to change the kernel during the boot.
### I'm Using Purecaps So I Can't Compile or Run Hybrid Code
No, that's not true. Both builds can run with either kernel and can run code built for either kernel.
### Some of My Code Doesn't Work When Built for Purecaps but Works on Hybrid so My Code Is Wrong!
Maybe not - this can happen for a number of reasons. Often, it is tempting to blame our code. I like to try my code out on a separate FreeBSD machine first and then I test it on Pure-Caps and Hybrid.
### I Can Compare Library Function Calls on hybrid and Pure-Cap Versions
Probably not. When the OS is built they build everything twice. There is a purecaps version and a hybrid version of the whole library. If a library function isn't working on purecaps, you can't just compare what you're passing in each case as the memory addresses will be completely different and any structs will by aligned differently for purecaps. In general, if you've found something that works in Hybrid but not Purecaps then you have found a bug. The bug may be yours but it may also be soemthing in the OS. If you're sure that the bug is not your mistake you can check on the Slack channel first and then report it on GitHub if it's confirmed.
### How Do I Upgrade CheriBSD?
There is no binary update mechanism on CheriBSD. What does this mean? It means the best way to update to the latest version is backup everything that you want from the machine and reinstall from scratch.
### Do I Always Get An Exception When I Go Outside My Capability's Range?
Yes and no. If you attempt to read or write beyond the end of the range you will get an exception. However, code can now also check to see if you were going to go outside of a valid area and return an error before you do. An example of this would be `read()` which range checks how many bytes it about to fill your buffer with and returns `EFAULT` if you're going to go outside the valid range. There is a small demo project that I created showing this [here](https://github.com/GrassHopper1977/CheriBSD_Example_Error). If you're unsure the comile both purecaps and hybrid versions of your code and compare them, that will give you a good idea of if it's a capabilities error or a regular error.

## How Do I use the Full-Caps Kernel?
You have to change the kernel when you get to the CheriBSD boot menu:
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
```
I press space to pause then press 6 until the Kernel: shows one of the purecap builds (I use `6. kernel: kernel.GENERIC-MORELLO-PURECAP (4 of 4)`).

You can also change the default kernel by adding the line `kernel="..."` to /boot/loader.conf. The name must be a subdirectory of /boot that contains a kernel. e.g kernel, kernel.GENERIC-MORELLO-PURECAP, etc.

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


# Compiling on CheriBSD
## Notes
- GCC hasn't been built for CheriBSD yet so they are using `clang`.
- There is a shell script called `cc` that builds the code for which ever ABI you are currently using. So on the Pure-caps build it will default to building pure-caps, on the hybrid build it will build hybrid code.
- You can override and chose to build for either ABI from either OS build and using any of the kernels available.
- Capabilities give you extra things to think about. If you're using a lot of structs then you may wish to include the additional `-cheri-bounds=subojbect-safe` option (TODO: explain why).
## Compiling Code
As mentioned the `cc` shell script builds for which ever OS build you are currently using. So if you execute:
```
cc -g -O2 -Wall -o hello hello.c
```
We can force it to build one way or another though. To force it to build in Hybrid mode:
```
cc -g -O2 -Wall -mabi=aapcs -o hello hello.c
```
To force it to build in Pure-Caps mode:
```
cc -g -O2 -Wall -mabi=purecap -o hello hello.c
```
I have found it useful to create a shell script to build both version at the same time. This is a my script to build both versions of some code I wrote to test libusb:
```
cc -v -g -O2 -Wall -mabi=aapcs -cheri-bounds=subobject-safe -lusb  -o testusb_hy testusb.c
cc -v -g -O2 -Wall -mabi=purecap -cheri-bounds=subobject-safe -lusb  -o testusb_pc testusb.c
```
It's nothing complicated but I'll break down the commands here:
- `cc` - the compiler shell script
- `-v` - Verbose output (I find it useful but its' not essential)
- `-g` - Generate debug information
- `-O2` - Optimisation level. This is a moderate optimisation level. See [here](https://www.freebsd.org/cgi/man.cgi?query=clang&sektion=1) for more details on this option.
- `-Wall` - Display all warnings
- `-mabi=` - Is used to set the Application Binary Interface (for this machine it's `aapcs` for Hybrid mode and `purecap` for Pure-Caps mode)
- `-cheri-bounds=subobject-safe` - By default a sub-object within a struct would have the same boundaries as the struct, meaning that it is possible to write outside the area of the object. This option prevents that. For example imagine a struct contain an array of 4 chars followed by a uint16_t. A pointer to that struct would read the same as a pointer to the first item in the struct (in this case the array of 4 chars). By default, the capabilities would be the same, meaning that you could write beyond the end of the array and into the following uint16_t (but not beyond the end of the struct). This option would ensure that there are separate capabilties for the pointer to the struct and to the first item within the struct.
- `-lusb` - Include the libusb library
- `-o testusb_pc` - Sets the name of the output file to `testusb_pc`
- `testusb.c` - Source files.

## nano does have code folding or syntax highlighting!
You can use sshfs to mount/map a local directory/drive to the drive on the using SSHFS [see our instructions on setting this up on CheriBSD here](sshfd.md).


## Understanding CHERI Code and CheriBSD (a CHERI tutorial)
I highly recommend that you go through this tutorial. It's not written for Morello CHERI but you can still try the exercises by comiling for Hybrid or Purecaps. It will give you a good understanding of what CHERI can (and cannot) do: [Adversarial CHERI Exercises and Missions](https://ctsrd-cheri.github.io/cheri-exercises/cover/index.html)

# Trying out a Custom Kernel
1. Navigate to our home directory: `cd ~`
2. Use `wget` to get the tar file onto our machine: `wget https://www.cl.cam.ac.uk/~jrtc4/kernel.UGEN-TEST.tar.xz`
3. Navigate to the root of the file system: `cd /`
4. Now we need to unpack it into the root of our install: `tar -xf ~/kernel.UGEN-TEST-tar-xz`
5. Now we can try rebooting and loading the new kernel, which should be called 1kernel.UGEN-TEST`
6. Enter `shutdown -r now` (don't forget to unplug any USB devices before rebooting)
7. Wait for the CheriBSD Boot Menu:
```
|   _____ _               _ ____   _____ _____
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
 |  6. Kernel: default/kernel (1 of 5)     |      --                      -.
 |  7. Boot Options                        |       `:`                  `:`
 |                                         |         .--             `--.
 |                                         |            .---.....----.
 \-----------------------------------------/
```
8. Press space to stop the countdown.
9. Press 6 until the kernel option reads: `6. Kernel: kernel.UGEN-TEST (5 of 5)`
10. Now press enter to continue the boot sequence.
11. Log in
12. Use uname to check that we are running the new kernel:
```
root@cheribsd:~ # uname -a
FreeBSD cheribsd.local 14.0-CURRENT FreeBSD 14.0-CURRENT #0 ugen-ep-copyincap-n256372-c708375e636: Mon Jan  9 16:16:12 GMT 2023     jrtc4@technos.cl.cam.ac.uk:/local/scratch/jrtc4/libusb-cheribuild-root/build/cheribsd-morello-purecap-build/local/scratch/jrtc4/libusb-cheribuild-root/cheribsd/arm64.aarch64c/sys/GENERIC-MORELLO arm64
```
So this is a purecaps build.
13. Go and test to see if it it fixes the issue that you've been having.

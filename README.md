# Morello CHERI Setup
My notes on setting up the Morello CHERI board with CheriBSD, upgrading it and various things I've noticed.

# Initial Setup and Install
## Things You Need to Know!
- Some instructions come with the Morello Cheri board. They are pretty confusing and will make your life difficult. For those of us on the DSbD program, we are suopposed to be trying to create things with the Morello Cheri rather than actively contributing to the development of it and building the OS, etc. We are better off using one of teh pre-built OS images to start with.
- There are three pre-built OS available:
 - CheriBSD - A CHERIfied version of FreeBSD. This is the most developed, most complete and the only one I've played with.
 - Android - I haven't tried this
 - Linux - I haven't tried this either.
- I used the guide [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/cover/index.html) and added my own notes as I went along to create this.

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
Cmd reboot
Rebooting...
Disabling debug USB..
```

## Updating CheriBSD
We follow the basic instructions from [here](https://ctsrd-cheri.github.io/cheribsd-getting-started/getting/index.html) but with a little more detail.

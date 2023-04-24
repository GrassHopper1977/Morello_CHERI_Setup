# Building CheriBSD

Various ways to build CheriBSD.

## Sources
https://github.com/CTSRD-CHERI/cheribuild
https://ctsrd-cheri.github.io/cheri-exercises/introduction/cross-compilation-execution.html
https://github.com/CTSRD-CHERI/cheripedia/wiki/HOWTO:-Build-CheriBSD-natively-on-Morello

## Building on Ubuntu
I'm using Ubuntu V18.04.6 LTS and cross compiling it for the Morello CHERI board. 

Note: I had to update my version of cmake follwing these instructions: https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line
### Installing Pre-requistes and Configuring
1. Fetch the prerequistes: `sudo apt install autoconf automake libtool pkg-config clang bison cmake mercurial ninja-build samba flex texinfo time libglib2.0-dev libpixman-1-dev libarchive-dev libarchive-tools libbz2-dev libattr1-dev libcap-ng-dev libexpat1-dev libgmp-dev'
2. Clone CheriBuild: `git clone https://github.com/CTSRD-CHERI/cheribuild.git`
3. `cd cheribuild`
4. Now we'll try to build CheriBSD for Morello in Purecap: `./cheribuild.py cheribsd-morello-purecap -d`
5. The build will take a few hours as it's a full build (You may want to use incremental builds for trying things out and reserve full builds for once you think it's working correctly)
6. Assuming this works, the followig will happen:
* Any sources will be stored in: `/home/[USER]/cheri`
* The output files will be stored in: `/home/[USER]/cheri/output`

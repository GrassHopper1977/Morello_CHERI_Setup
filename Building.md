# Building CheriBSD

Various ways to build CheriBSD.

## Sources
* https://github.com/CTSRD-CHERI/cheribuild
* https://ctsrd-cheri.github.io/cheri-exercises/introduction/cross-compilation-execution.html
* https://github.com/CTSRD-CHERI/cheripedia/wiki/HOWTO:-Build-CheriBSD-natively-on-Morello

## Building on Ubuntu
I'm using Ubuntu V18.04.6 LTS and cross compiling it for the Morello CHERI board. 

Note: I had to update my version of cmake follwing these instructions: https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line

### Installing Pre-requistes and Configuring
1. Fetch the prerequistes: `sudo apt install autoconf automake libtool pkg-config clang bison cmake mercurial ninja-build samba flex texinfo time libglib2.0-dev libpixman-1-dev libarchive-dev libarchive-tools libbz2-dev libattr1-dev libcap-ng-dev libexpat1-dev libgmp-dev'
2. Clone CheriBuild: `git clone https://github.com/CTSRD-CHERI/cheribuild.git`
3. `cd cheribuild`
4. Now we'll try to build CheriBSD for Morello in Purecap: `./cheribuild.py cheribsd-morello-purecap -d`
5. The build will take a few hours as it's a clean build. It will automatically clone the `main` branch of the [CheriBSD repository](https://github.com/CTSRD-CHERI/cheribsd) and then build the code.
6. Assuming this works, the followig will happen:
   * Any sources will be stored in: `/home/[USER]/cheri`
   * The output files will be stored in: `/home/[USER]/cheri/output`

### Performing an Incremental Build
Since it takes several hours to perform a clean build so this may not be useful all the time. An incremental build may be more useful if we just want to make simple changes to the kernel to try them. You'll note that we don't use `-d` as that means "build all dependencies" which may make the build longer. The command I use is:
```
./cheribuild.py cheribsd-morello-purecap --no-clean
```

### Building a Different Branch
NOTE: This is returning an error at present. We're still investigating why.
1. After performing the initial build, you can use `git` to switch to another branch. In this example we'll switch to `dev` (which is where most of the work is done and we currently need in order to get the USB fix that was fixed [here](https://github.com/CTSRD-CHERI/cheribsd/pull/1619/commits/c708375e636a6a80468eda19a9beebe9c1d618cb)). At the time that this document was written, `dev` was over 10000 commits ahead of `main`. This gives you an idea of howimportant teh `dev` branch is for getting the latest features and fixes.
2. Change to your local source folder: `cd /home/[USER]/cheri/cheribsd`
3. You can obtain a list of exisitng local branches using the following (first time it only show `main` as an available branch):
```
git fetch --all
git branch
```
4. Switch branch using to `dev`:
```
git checkout dev
```
5. We should now be using the `dev` branch. If you're careful (or paranoid) you can use `git log` to double check that you have the expected commit by comparing the ID with the one on the CheriBSD github page.
6. Now we may want to build this branch (we may also want to include the dependancies). Remember that the `cheribuild` command will need to be executed from the `cheribuild` directory. We're only going to build teh items that have changed:
```
./cheribuild.py cheribsd-morello-purecap --no-clean
```

# Building CheriBSD

Various ways to build CheriBSD.

## Sources
* https://github.com/CTSRD-CHERI/cheribuild
* https://ctsrd-cheri.github.io/cheri-exercises/introduction/cross-compilation-execution.html
* https://github.com/CTSRD-CHERI/cheripedia/wiki/HOWTO:-Build-CheriBSD-natively-on-Morello

## Notes On CheriBSD Repository Branches
CheriBSD is based on FreeBSD which has been around for years and has moved through several different Version Control Systems. Consequently, it has a slightly non-obvious structure. In most git repositiories, new features would be added on branches which would then be merged into main and a Tag would be made for each new release. CheriBSD is slightly different.
* Development takes place in new branches.
* Working features are merged in the `dev` branch and the feature branches are deleted.
* For a release, a tag would still be created and that is based on the `main` branch.
* Releases are created on a `releng` branch (perhaps from "release to engineering"?). For example, `releng/22.12` is the release branch for V22.12 which may be different from the version on `main`.
* The `releng` branch may be updated multiple times with teh new releases organised by date.
* This means that the `releng` branch may be ahead of both `main` and the Tag upon which it was originally based.
* Certain features can be cherry picked from the `dev` branch and merged into `releng` branches to fix issues as they appear.

## Building on Ubuntu
For this example I'm using Ubuntu V18.04.6 LTS and cross compiling it for the Morello CHERI board. 

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
Note: We're still having an issue building this. I'm working on it.
1. After performing the initial build, you can use `git` to switch to another branch. In this example we'll switch to `releng/22.12` (which is the current release). 
2. Change to your local source folder: `cd /home/[USER]/cheri/cheribsd`
3. You can obtain a list of exisitng local branches using the following (first time it only show `main` as an available branch):
```
git fetch --all
git branch
```
4. Switch branch using to `dev`:
```
git checkout releng/22.12
```
5. We should now be using the `git checkout releng/22.12` branch. If you're careful (or paranoid) you can use `git log` to double check that you have the expected commit by comparing the ID with the one on the CheriBSD github page.
6. Now we may want to build this branch (we may also want to include the dependancies). Remember that the `cheribuild` command will need to be executed from the `cheribuild` directory. We're only going to build teh items that have changed:
```
./cheribuild.py cheribsd-morello-purecap --no-clean
```

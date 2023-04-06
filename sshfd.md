# Introduction
CheriBSD's nano doesn't have syntax highlighting and doesn't support code folding. You may wish to use another editor on a different machine. The easiest way is to use SSHFS to map the home directory of you CheriBSD user to your local machine. Then you can use your preferred IDE. SSHFD should already be a part of your SSH service on CheriBSD so we just need to connect to it from our remote machines. This, obviously, depends on which OS you are using.

# Linux
Note: This was tried on Ubuntu. SSHFS is supported on most Linux distros.
1. From the command line:
```
sudo apt update
sudo apt install sshfs
```
2. Now we need to create a subdirectory to mount to.
```
sudo mkdir /mnt/morello
```
3. Now we use `sshfs` to mount to that directory. Note: The use of `~/` to specify the user's home directory doesn't seem to work. You have to specify the full path to the directory as shown here for teh root user (if your user was called `bob` your path would be `/home/bob/`). Obviously, substitute `root` and `server_name` for appropriate ones for you:
```
sudo sshfs -o allow_other,default_permissions root@server_name:/root/ /mnt/morello
```

# Windows 11
There is a port of SSHFS for Windows that can be found [here](https://github.com/winfsp/sshfs-win). I was unable to get it working with Windows 11 but others may have better luck.

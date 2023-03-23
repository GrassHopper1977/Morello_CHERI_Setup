# Introduction
CheriBSD's nano doesn't have syntax highlighting and doesn't support code folding. You may wish to use another editor on a different machine. The easiest way is to use SSHFS to map the home directory of you CheriBSD user to your local machine. Then you can use your preferred IDE. SSHFD should already be a part of your SSH service on CheriBSD so we just need to connect to it from our remote machines. This, obviously, depends on which OS you are using.

# Windows 11
1. Install WinFsp · Windows File System Proxy (https://github.com/winfsp/winfsp/releases/tag/v2.0)
2. Install SSHFS For Windows (https://github.com/winfsp/sshfs-win/releases)
3. Open `Windows Explorer` and right-click on `This PC`
4. Select `Map Network Drive`.![sshfs01](https://user-images.githubusercontent.com/52569451/227214510-9edee380-b8cc-4f57-8460-0ca97ad335e8.png)
5. Choose your drive letter and enter the folder path.
6. The folder path in our example is: `\\sshfs\root@192.168.1.181`.
7. Make sure to select `Reconnect at sign-in` and `Connect using different credentials`.![sshfs02](https://user-images.githubusercontent.com/52569451/227214551-cbb926cb-58d4-43fa-9db3-3490c150fdee.png)
8. Press `Finish` and enter your password when prompted.![sshfs03](https://user-images.githubusercontent.com/52569451/227214581-f84a867b-3329-4c78-9940-6671671d267c.png)

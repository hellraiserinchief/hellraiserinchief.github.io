---
title: "Install NPM alongside ROS Melodic"
date: 2019-05-04T01-46-00+5:30
categories:
  - blog
tags:
  - ROS
  - apt
  - npm
---


I have ROS installed on my Ubuntu Bionic (18.04) OS. When I try to install npm using apt `sudo apt install npm`, I get this wierd error:
```
kaustubh@Workbench:/etc/apt/preferences.d$ sudo apt install npm
Reading package lists... Done
Building dependency tree
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 npm : Depends: node-gyp (>= 0.10.9) but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```
According to[this git issue on ROS](https://github.com/ros/rosdistro/issues/19845), the error is related to different versions of nodejs dependency libssl-dev. [NickZ's](https://github.com/NickZ) comment told me what to do, but how? Also this [bug report](https://bugs.launchpad.net/ubuntu/+source/npm/+bug/1809828) confirmed the theroy that my error was indeed related to conflicting versions of.

I found the packages that had nodejs using, `sudo apt-cache policy nodejs`
```
nodejs:
  Installed: 8.10.0~dfsg-2ubuntu0.4
  Candidate: 8.10.0~dfsg-2ubuntu0.4
  Version table:
 *** 8.10.0~dfsg-2ubuntu0.4 500
        500 http://in.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages
        100 /var/lib/dpkg/status
     8.10.0~dfsg-2ubuntu0.2 500
        500 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages
     8.10.0~dfsg-2 500
        500 http://in.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages
```
I needed to shift the package source from 8.10.0~dfsg-2ubuntu0.4 to 8.10.0~dfsg-2ubuntu0.2 to fix the issue and install npm. This (link)[https://help.ubuntu.com/community/PinningHowto] helped a bit. So I set out to figure how to 'pin' my nodejs package.

FINALLY, found [this](https://www.howtoforge.com/a-short-introduction-to-apt-pinning), thanks Falko Timme for taking you time and writing this article.

I created a file called nodejs-pin under `/etc/apt/preferences.d` directory, with the following content:
```
Package: nodejs
Pin: version 8.10.0~dfsg-2ubuntu0.2
Pin-Priority: 1001

Package: nodejs-dev
Pin: version 8.10.0~dfsg-2ubuntu0.2
Pin-Priority: 1001
```
This gives the `8.10.0~dfsg-2ubuntu0.2` version priority of 1001, which is higher than 500 priority of `8.10.0~dfsg-2ubuntu0.4` version. I saved the file, ran `sudo apt update`, checked the pinned status using `sudo apt-cache policy`
```
.
.
.
500 http://in.archive.ubuntu.com/ubuntu bionic/main amd64 Packages
     release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,c=main,b=amd64
     origin in.archive.ubuntu.com
Pinned packages:
     nodejs -> 8.10.0~dfsg-2ubuntu0.2 with priority 1001
     nodejs-dev -> 8.10.0~dfsg-2ubuntu0.2 with priority 1001


```
Ran `sudo apt install nodejs nodejs-dev` and then `apt install npm`, which installed npm on my machine :)

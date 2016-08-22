---
layout: post
title: Unix usage at AUV IITKGP
author: Siddharth Kannan
github-username: icyflame
---

At AUV, we use Robot Operating System (Fondly called ROS) as the primary OS to run our robot. ROS is a meta-OS that runs on top of Linux. We chose Ubuntu from some of the flavors of Linux that ROS supports, simply because it seemed to have the most support for all the third party packages that we were going to be working with. We haven't regretted the choice of this setup, but there are a few things that I must say have gone awry since. We will touch upon them at the very end of this post.

First, let's start with the basics. The method we use to connect to our vehicle, and that's more or less how we get things done!

### How do we connect to our vehicle?

We use a 2-computer static IP setup. We have a special network, with the appropriate settings (It has the subnet mask of `255.255.255.0` and the gateway set to `0.0.0.0` and the required IP in the IP section inside network settings). So, this is how we connect to the vehicle:

1. Connect a LAN cord between the machine and the vehicle
2. Startup the vehicle
3. Connect to the AUV network on the machine. The vehicle is setup to connect to this network automatically on startup (if a LAN cable is attached, of course)
4. Now, a simple SSH command completes the process: `ssh username@192.168.10.19`. That's the IP we chose for the vehicle. (You should certainly think about the reason for that particular choice!)

Now, I agree that this setup might seem like something that is very adhoc and can be used only in a very narrow range of things, but is very helpful when you want to SSH between two machines, and you don't have an internet connection, or a router which can act as the DHCP server. We will talk more about the Unix utility `rsync` in detail, further down in the post, but I used rsync to transfer files between a Mac Mini and a Linux computer, because macOS can't write to NTFS file systems, and the only interoperable format (apart from `ext4`) was FAT32, which has a maximum file size of a painfully low [4 GB](https://answers.microsoft.com/en-us/windows/forum/windows_7-files/what-is-the-maximum-file-size-fat-fat32-ntfs-file/1663db6b-490e-4021-9e36-f7a6976ac0c0?auth=1). This was the exact setup that I used, and file transfer speeds were impressive at close to 15 MBPS, with a CAT6 cable connecting the two computers.

So, this setup is very useful whenever you want to connect two computers using SSH.

### How do we transfer code onto the vehicle?

Now, we are connected to the vehicle. We have access to a terminal on the vehicle. With [`tmux`](https://tmux.github.io/), a terminal multiplexer that we are un-endingly in love with, we have a lot of terminals, without having to open new SSH sessions everytime. This is particularly useful on any server that you might SSH into, instead of starting of processes with the `&` appended to the end and having no visiblity about them, [simply install `tmux`](https://github.com/icyflame/install-from-source-tmux-2-0) on your server, and off you go!

Now, we have a repository of code that keeps changing, and is one hell of a ROS module with many many packages on it, and often many of these packages are edited on the fly, on the vehicle's file system itself. So, we need a way to make sure that the code on the vehicle stays updated on the `master` branch at [our repository on GitHub](https://github.com/auviitkgp/kraken_3.0).

Now, to be honest, we have been facing issues with this, because it's really hard for everyone to be extremely systematic and take a backup of the code every time that vehicle is shut down or the testing session ends. So, onboarding the whole team onto this system is still a work in progress, but we will definitely get everyone on soon enough!

SO, onto the specifics, we use [`rsync`](https://en.wikipedia.org/wiki/Rsync), which is a simple utility to copy files over from one location to another, making sure it does the least copying possible by using file timestamps which are treated as first class citizens on Unix. `rsync` has an interface that depends heavily on the usage of forward slashes in the right manner to ensure that the folder structure is the one you want to. [rsync's man page](http://linux.die.net/man/1/rsync) is one of the man pages that explains everything, and I mean absolutely everything, with an example. So, it's a real fun man page to read and get help from whenever you are in a fix. This is the exact command that we use to sync up our code:

```sh
rsync -avzhe ssh --progress ~/ros_workspace/kraken_3.0 siddharth@192.168.10.11:~/kraken3`date +'%Y-%m-%d-%H-%M'`
```

picked out directly from the [vehicle's `~/.bashrc` file](https://github.com/auviitkgp/dotfiles/blob/d5037851c1d255c4a0f81720d61a37dafb6ca05b/bashrc#L153), which is a part of the [auviitkgp/dotfiles](https://github.com/auviitkgp/dotfiles) repository which has the latest vehicle dotfiles on it. Check it out for [a few other](https://github.com/auviitkgp/dotfiles/blob/d5037851c1d255c4a0f81720d61a37dafb6ca05b/bashrc#L94-L118) not-so-interesting aliases that make life simpler for sane people.

Needless to say, `rsync` is a great utility both when working with remote servers and when working with two separate partitions/folders on your local machine / flash drive / external hard drive, etc etc. The use cases for this small utility are endless. A few special mentions about `rsync`. The `--progress` flag: to show graphical progress of the file transfer on the terminal. The `--partial` flag: which ensures that if you are copying really large files and are afraid of losing the connection, then rsync will keep your partially downloaded file and patch it the next time you have network and download only what's remaining and not the whole file again.

So, next time, before you `cp`, `rsync`.

That's all for this post. We will follow this post up with other posts about the other sideswipes that were necessary to make startup scripts work (because `upstart` isn't something that can run environment dependent bash files, and `init.d` is a mammoth without any good documentation/example sets/quality SO questions yet).

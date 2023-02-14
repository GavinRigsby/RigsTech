---
title: Running Windows 10/11 in Linux with LXD
---

Getting VM's is useful for a myriad of different things. Yeah you could use VMware or another Virtual Machine host but what if you wanted to automated the process and not have to manually setup the VM. Today I am going to show you how to simply create Windows hosts in LXD with minimal setup.

[LXD](https://linuxcontainers.org/lxd/introduction/ "") (Linux Container Daemon) is a system container manager for Linux that provides an experience similar to a virtual machine, but uses containers instead of virtual machines to run applications. With LXD, you can create and manage system containers, which are isolated environments that provide a secure and efficient way to run applications and services. You can use LXD to host applications, run databases, and even create multiple isolated development environments, all within a single physical server.

You need to have LXD already running and configured. If you still need to get that setup go checkout my LXD setup page [here](http://localhost:3003/Setting-up-LXD "here").

There are a few other prerequisite tasks to do before we can jump right in.

1. Get a Windows 10/11 ISO
2. Use distrobuilder to prepare the ISO
3. Start the VM and go through install steps

## Getting a Windows ISO

Since Microsoft doesn't publicly release ISO files we will have to use a tool to get it.

This tool requires you have Windows 10/11 previously installed on a computer.

Download the [Windows 10](https://www.microsoft.com/en-us/software-download/windows10 "")
or [Windows 11](https://www.microsoft.com/en-us/software-download/windows11 "") Media Creation Tool.

Run the tool and after the inital setup it will prompt you to Accept the EULA.

![Windows Media Creation Tool|50px](/Win10MediaCreation.PNG "Select 'Create installation media' to create ISO")

Select the option to create Installation media and keep the default settings.
Select the ISO file option and save the file to your computer.

This will take some time to download but once finished you can transfer the file to your linux computer and continue to the next section.

## Use distrobuilder to prepare the ISO

Use the following commands to install distrobuilder with snap and a few dependencies

```
sudo snap install distrobuilder --classic
sudo apt install -y libguestfs-tools wimtools
```

Once installed we will run distrobuilder on our ISO file (ISO names might differ slightly)

```
sudo distrobuilder repack-windows Windows10.iso Windows10-build.iso
```

If you get an error detecting windows version add `--windows-version <version>`

where \<version> is w10 for Windows 10 and w11 for Windows 11

This will generate the Windows10-build.iso and took me around 5 minutes

## Install Windows

We are going to run the following commands to configure a VM with 30GB of storage and safe boot off.
Note: The install takes around 10GB before updates giving around 20GB of free space.

```
lxc init win10 --empty --vm -c security.secureboot=false
lxc config device override win10 root size=30GiB
lxc config device add win10 iso disk source=/path/to/Windows10-build.iso boot.priority=10
```

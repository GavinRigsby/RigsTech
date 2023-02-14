---
title: Create an Automatic Windows Install (Unattended Installation)
---

This guide will show you how to create an automated installation of Windows 10 or Windows 11, just follow the steps below.

When you start a new Install of Windows you must go through the setup process. This process lets the user configure various settings like language preferences, product key, and partition layout. After the installation you will go through the out-of-box experience (OOBE) to configure settings like keyboard layout, account information, and privacy settings.

The process is relatively simple however if you have to install Windows on multiple workstations this may take up a significant amount of time. The answer to this problem is to use the Unattended Installation process.

To automate the install you can create an answer file that contains the instructions to automatically complete the on-screen setup. This is integrated into the boot media (USB, cdrom, PXE, etc.) and is read to install Windows automatically.

This guide will walk you through the steps to create the `autounattend.xml` answer file to do a basic unattended install of Windows.

## The Process

There are many wants to create the automated install process. This guide outlines how to create the autounattend.xml for unattended installations of Windows 10 Pro on a 64-bit computer running UEFI or BIOS.

Like any other ISO the install process will erase everything on the drive and install of Windows 10.

NOTE: If you have important information on the drive you are wanting to install the Windows unattended install on then create a full backup before proceeding.

## Requirements

Some things you will need to complete the process.

* Windows Assessment and Development Kit (ADK)
* Windows 10 ISO
* Windows 10 computer with admin privileges
* Computer or VM to test the installation (VM preferred)&#x20;
* USB with 8GB of space (If using computer)

## Install Windows System Image Manager (WSIM)

You can write the answer file manually however Microsoft has the Windows System Image Manager (SIM) console though the Assessment and Development Kit (ADK) to help create the file.

To install the Windows System Image Manager, do the following:

1. Download the correct version of the [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install "Windows ADK")

(In this tutorial we will be using the ADK for [Windows 11, version 22H2](https://go.microsoft.com/fwlink/?linkid=2196127 ""))

1. Run the adksetup.exe to start the install
2. Select the option to Install **to this computer**
3. Click **Next > Next > Accept** to specify the location, specify privacy, and accept the EULA ![Install Deployment Tools](/DeployementToolInstall.PNG "Install Deployment Tools")
4. Click **Install** to install the Deployment Tools
5. Once the install is finished click **Close**

## Create Windows 10 answer file project

After installing Windows SIM you need to import the ISO file and setup the environment to create an answer file.

### Import Windows 10 Image

In order to create an answer file, you need to open a Windows ISO and create a catalog of all the components to automate the installation. First we must import the Windows 10 install files using the steps below:

1. You need to have a Windows ISO, if you don't already you can use the [Media Creation Tool](<Media Creation Tool> "https://go.microsoft.com/fwlink/?LinkId=691209") or [Windows ISO Downloader](https://www.heidoc.net/php/Windows-ISO-Downloader.exe "Windows ISO Downloader") by heidoc.net to get an ISO.
2. Navigate to the ISO file location and right click the ISO file.
3. Select the Mount option to mount the media to your computer.
4. Open the mounted drive and copy all files.
5. Create a new folder somewhere on your computer (Desktop, Documents, etc.) and paste the files.
6. Unmount the ISO by right clicking the mounted drive and clicking Eject.

Once these steps are finished we have all the install files saved to our computer. We need to confirm that the **install.wim** file is present in the 'sources' folder. If you used the Media Creation Tool you likely have a install.esd instead of install.wim because this file is encrypted.&#x20;

If you have an install.esd file and not a install.wim then we will need to convert the file to continue.

#### Convert install.esd to install.wim (skip if install.wim is present)

To convert the .esd file into a .wim we need to use the DISM command line tool in powershell to export the image files.&#x20;

1. Open Start Menu
2. Search PowerShell, right click and select Run as administrator
3. Type the following command to get the index number of the edition you want to use. `dism / Get - WimInfo / WimFile: C: \path\to\folder\sources\install.esd`
4. Below each index is a windows version available in the ISO (we will choose 6 for Windows 10 Pro)
5. Enter the following command replacing \<path> to the path of your folder you created in the Windows 10 Image task and \<index> with the index you selected. `dism /Export-Image /SourceImageFile:<path>\sources\install.esd /SourceIndex:6 /DestinationImageFile:<path>\sources\install.wim /Compress:Max /CheckIntegrity`

This will create a install.wim file in the sources folder of the Windows Installation Files.

Note: If this doesn't work don't worry just download a Windows ISO with the install.wim file using the [Windows ISO Downloader](https://www.heidoc.net/php/Windows-ISO-Downloader.exe "Windows ISO Downloader") by heidoc.net

#### Setup an answer file enviornment

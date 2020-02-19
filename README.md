# lk_bootloader_lumia640
prebuit LK shim from @imbushuo a long time ago.


How to install LK bootloader on Lumia Models newer than x2x
By: sjrmac
WRITTEN initially in May of 2018

Background

	To start off, this process is not nearly like the x2x installation but I’m sure it will improve over time. The real mastermind behind all of the unlocking of the bootloader through the program known as Windows Phone Internals, is HeathCliff74XDA. He provided us with the original unlock of Lumia x2x models, allowing mass storage and the ability to completely remove the WP operating system from those models and replace the bootloader. With this new unlock type, known as “Lumia Bootloader Spec B”, the most access we are allowed is a partial bootloader unlock. We retain the ability to access mass storage, but cannot remove the WP bootloader or required partitions. This is where developer imbushuo came in. He figured out that a boot-shim could be created in order to chainload things we needed through the current UEFI WP bootloader. He then created “EDK2 Implementation for Lumia 950 XL” which sparked heavy interest in his work at the time. He showed that it was possible to run a AArch64 UEFI on a device that shipped with a 32bit implementation. During this, Ben(imbushuo) was working on a modified LK that “bootstraps Android/UEFI on Lumia 930 and 950 XL”. I got in contact with Ben and started chatting about getting a Lumia 640 base up to speed. He started working on it and now we have something that can be installed so far. (5/23/2018)

Installation

	So, to begin, let’s start here as you need to know what is required to get this working.

Prerequisites:  
A Windows PC running Windows 7 or newer with Admin Privilege  (preferably newer)
Windows Device Recovery Tool installed (good for drivers and restoring from bricks)
Windows Phone Internals 2.4 and newer
ADB and Fastboot drivers (I use 15sec adb and fastboot installer from XDA)
Win32DiskImager
A internet connection to download firmware through WP Internals and occasional Windows drivers.
A currently supported Lumia model or rather MSM model (cpu- check GSMArena specs)
Some knowledge of running commands in CMD or Powershell.
Patience for when shit isn’t going your way.

	To begin, make sure you have fulfilled all of the prerequisites before beginning. After that is complete, go ahead and visit https://github.com/imbushuo/lk to get Ben’s LK sources that you need to build IF a precompiled version isn’t available. If you do know how to compile from source on a Linux distro, go ahead and build for the branch that matches your Lumia’s msm model. Once that is complete if you have built, check your “build-<msm>/” inside the LK source tree and grab the file “lk_s.elf”. After you have “lk_s.elf”, you should go and download the latest boot-shim available from Ben’s GitHub here: https://github.com/imbushuo/boot-shim/releases
All you need from the release tab is the latest version of the “ELF Variant” and the file from that release should be named “BootShim.efi”. At this point, you have both of the needed files, the LK build and the boot-shim. Go ahead and make a folder on your PC in a location you can remember and name it like “Lumia Android files” or something.

	If you have not already, it would be a perfect time to open Windows Phone Internals 2.4 (or newer) and connect your Lumia. Go to the tab named “Info” and wait for your device to appear after plugging your phone. Once it appears, go to the “Download” tab. Under the “Model” section you can find that your device info such as “Producttype” has already been filled in. Click the “Search” button and wait a moment. WP Internals will pull up all required files that it needs to download in order to unlock your device. Click download all and wait for it to finish up as this may take some time depending on your internet speed. Once completed, go to the “Unlock Bootloader” tab and click “OK” to switch your phone to flash-mode. A page will appear and you can go ahead and start the unlock process. Wait for WP Internals to finish and it should ask you to reboot your phone so complete that.

	Now that our bootloader is unlocked, we can move forward. At this stage, it is a good idea to backup the eMMC of your device in case something happens. In WP Internals, select the tab “Manual mode” and then reboot your device to “Mass-storage mode”. Once your device is in mass storage mode, a popup may appear asking you to format your device’s mounted disk. DO NOT FORMAT IT. /*Unplug all disks except your Lumia*/ Open “Win32DiskImager” and select the the “Drive Letter” of your device. You can check for that in File Explorer under “This PC”. It will be titled “MainOS” and the drive letter will be next to the name. After you have selected your device’s disk inside of Win32DiskImager, use its file path selector to link to a folder you want to store the backup file in. Give it a name in the file selector bar, something you can remember for your device, or if you have multiple, label them per IMEI or something. EACH BACKUP IS UNIQUE TO ITS DEVICE! Now, click “Read” and the phone will start backing up to the image file where you wanted it to. Make sure to let the backup complete and not interrupt it while it is running. DO NOT DISCONNECT the phone at any point during the process. You may eject the drive after the backup is completed after you confirm file size in File Explorer if you would not move forward. Keep the phone connected.

	Here’s the fun part. We need the files we downloaded earlier, the “lk_s.elf” and “BootShim.efi”. Open a new File Explorer window and navigate to the “MainOS” disk aka your phone. Now, open another File Explorer window that has those two downloaded files for the boot-shim and LK. Rename “lk_s.elf” to “emmc_appsboot.mbn”. In the window with “MainOS” open, open the folder called “EFIESP”. Copy “emmc_appsboot.mbn” and “BootShim.efi” to that folder and don’t do anything else, leave them there. Now, open Windows Powershell and run it with Administrator privileges. From here, you will need to use the “cd” command to change your directory to the phone’s “MainOS” disk.
**Command: “ cd (driveletter):\ ” then “ cd EFIESP\efi\Microsoft\Boot ”, run “dir” and make sure BCD is there. Now, let’s check our BCD list. 
**Command: “ bcdedit.exe /store .\BCD /enum ALL ”
Now scroll up and find the text “ffuloader.efi”, above that should be a identifier string or also known as “GUID”. Copy that identifier string from { to } and paste it in a text editor. Run this next: “ bcdedit.exe /store .\BCD /set "{guid}" "nointegritychecks" 1 ” and replace “guid” with the actual string and run the command. In this case, we have replaced the “ffuloader” because the device I used for this (640) does not have a enter button. As developer Feherneoh puts it, “we are changing the path and disabling signature checks”. Make sure all your commands have run.

	The last bit is here! Yay! Go ahead and eject your phone from your PC then hold Volume down and power for about 10 seconds to reset/reboot the Lumia. Once the phone vibrates, hold Volume up and wait for the Microsoft logo to disappear. You should now be in LK based on the messages on the screen.

# How to Internally Flash ***coreboot*** to 2010-2013 Intel MacBooks via IFD Exploit

## By: Adam Wakelin (@atom-wakelin)

## Last Updated: 3/27/2026

*Special thanks to Evgeny Zinoviev (@gch1p) and the dedicated authors of the Coreboot project. This would not be possible without their documentation and expertise, which was an essential resource in my research and development.*

## ⚠️ ***DISCLAIMER:*** I am an independent author that is not affiliated with the Coreboot Project. (Maybe one day) I have taken the time to research and develop this guide using resources originally written by the Coreboot Project's authors and through testing on my own hardware. I am not responsible for any damage to devices that have undergone this procedure. Use at your own risk. ⚠️

>## ***Helpful References:***

>[Coreboot FAQ (What is Coreboot and what does it do?)](https://www.coreboot.org/end_users.html)

>[gch1p's original IFD hack (2019)](https://ch1p.io/coreboot-macbook-internal-flashing/)

>[gch1p's "On the current state of coreboot on MacBooks" (2023)](https://ch1p.io/coreboot-macbook-support/)

>[UEFI Secure Boot Explained](https://tianocore-docs.github.io/Understanding_UEFI_Secure_Boot_Chain/draft/secure_boot_chain_in_uefi/uefi_secure_boot.html)

>[My Hackaday project page](https://hackaday.io/project/204549-flashing-coreboot-firmware-to-intel-macbooks)

>[My GitHub Research and Development Repository (bleeding edge, most up-to-date)](https://github.com/Atom-Wakelin/Intel-Based-MacBook-coreboot-Research-And-Development)


# A Brief Introduction / Purpose

Before we begin, let me introduce myself. My name is Adam Wakelin, an avid GNU/Linux user and FOS enthusiast from San Diego, California, United States. I am passionate about protecting computer security at the software, firmware, and even hardware level.

## ❌ The Issue:

Every computer has ***hardware,*** (Keyboard, Screen) ***software*** (Windows, GNU/Linux, MacOS) and inbetween, there is ***firmware***, which acts as a middleman between both. ***Firmware*** tells the hardware how to run the software, and the software how to run on the hardware. All three play an essential role in the functioning of a computer.

Unlike hardware and software, users have very little control over their computer's firmware. Computer manufacturers actively obfuscate (hide) the source code and functions of their firmware from end users, and design computers to only run the firmware that the manufacturer approves of.

***The end user can neither see what the firmware is doing, nor modify it without bricking their system.***

Because of this, users of modern computers have no choice but to blindly trust that the firmware they have been forced to run is secure and has their best interests in mind. This is especially important for individuals or contractors that handle sensitive data and need to ensure that there are no security vulnerabilities in their systems that would make them vulnerable to external attackers.

## ✅ The Solution:

*Coreboot*, (or "coreboot" when spelled in lowercase, as is preferred by its authors) is Free and Open-Source (FOS) firmware that serves as a replacement for ***proprietary factory firmware.***

Unlike proprietary factory firmware, end users have full access to the source code and the ability to modify/configure it in a multitude of ways. Users know exactly what it does, how it does it, and can vet the source code for themselves.

Coreboot is designed be lean, lightweight, and to accomplish the bare necessities of a system boot before passing control of the host machine to a traditional BIOS/UEFI. Coreboot only accomplishes the ***necessary*** parts of boot, without the bloat.

There is a caveat: it aims to replace *as much firmware as **POSSIBLE***, aside from necessary proprietary pieces of firmware referred to as "blobs", which, due to the way manufacturers have designed their machines, most systems will not boot without. Because of this, sometimes, when we compile Coreboot, (like baking a cookie from a recipe) we sprinkle in the proprietary firmware blobs we need for the system to boot. (like adding chocolate chips) We can make our own cookie dough, (Coreboot) but we still need to add in elements we CAN'T replicate ourselves. (chocolate chips / Proprietary blobs)

At the end of the day, the goal of Coreboot is to replace as much proprietary firmware ***as possible*** to give the end user as much control over their firmware as possible.

***⚠️ Something to keep in mind: Coreboot does not work on all systems, and is becoming more difficult to implement on modern hardware that is designed to self-brick when modified. Luckily, there are dozens of older computers that support Coreboot, some of which we will be using in this guide: ⚠️***

# So, Why Use 2010-2013 MacBooks? (🍎 Bad Apples 🍎)

Most modern computers have firmware read/write protections implemented by their manufacturer to prevent users from modifying critical system data. Apple, like other computer manufacturers, also took steps to include read/write protections on their firmware.

Between 2010-2013, however, Apple manufactured a handful of MacBook models that all share a common vulnerability in their design that we can exploit as an attack vector to write Coreboot to our chip. 

For some reason, these models, due to a bug in their factory firmware, have their write protections disabled when they are activated from a ***cold boot*** state.

I like to refer to the affected models from this time period as ***"Bad Apples".***

***Bad Apples* are good candidates to run Coreboot for a couple of other reasons:**

1. **No extra hardware is needed** - They can be ***Internally Flashed.*** ***Internal Flashing*** is the Coreboot "installation" process that this guide utilizes. (We can put GNU/Linux on our target machine, plus a USB to store our firmware backups, and perform the entire process from the command line on the target machine itself. No extra computers, clamps, or adapters needed.)
2. **They have good hardware** - Older MacBooks are built very well and are reliable, long-lasting machines. Many can perform just as well as they did nearly 2 decades ago once GNU/Linux is installed.
3. **They are easy to acquire** - As they are older models, they can be found on sites such as eBay for around $100.00 USD or less. They are very cheap, easy to find, and have thousands of spare parts available for sale. **Think: "Ship of Theseus."**

# The MacBook IFD Exploit Summary of Steps:

***During this guide, we will be performing the following operations:***

1. Back up entire factory firmware to .bin file.
2. Extract regions from .bin file as a layout in a .txt file.
3. Split .bin backup into 3 .bin files, one for each region.
4. Truncate (Shrink) Intel ME .bin file.
5. Write a new layout .txt file with new region parameters.
6. Extract regions from new layout .txt file as .bins, one for each region.
7. Configure temporary Coreboot ROM, write to chip.
8. Cold boot (Reboot won't work)
9. Write final layout .txt file
10. Configure full-size Coreboot ROM
11. Flash full Coreboot ROM
12. Fully Corebooted MacBook!

# What You Will Need:

## 1. A USB for storing your files (FAT32, exFAT...)
## 2. GNU/Linux Bootable Drive
## 3. An Internet Connection
## 4. A corresponding charger for your MacBook
## 5. A 2010-2013 MacBook of the following models:
#### 💻 MacBook Pro 8'1 (Model #A1278)(✅ Internal Flashing TESTED AND CONFIRMED TO WORK)
#### 💻 MacBook Air 4,2 (Model #A1369)(❌ Internal Flashing UNTESTED but should work)
#### 💻 MacBook Air 5,2 (Model #A1266)(❌ Internal Flashing UNTESTED but should work)
#### 💻 MacBook Pro 10,1 (Model #A1398)(❌ Internal Flashing UNTESTED but should work)

>***⚠️ If your computer has soldered RAM, you will need to make sure your RAM is supported. Please refer to @gch1p's GitHub page [HERE](https://github.com/gch1p/mmga?tab=readme-ov-file#ram-configurations) to verify that your RAM is compatible. ⚠️***

*Please let me know on my GitHub or Hackaday page if you successfully flash Coreboot to an untested model so that I can port this guide to it.*

# Preparing your MacBook:

1. Plug your MacBook into a source of power. (*STEADY* power source, like a wall outlet)

2. Boot from a GNU/Linux drive. (GNU/Linux on the internal hard drive or on a bootable USB stick) I will be using a Debian-based distribution, so Shell commands may vary.

3. Plug in your USB storage device, which you will use to store all of your backups and will serve as your working directory.
   
4. Connect to the Internet (Used to build utilities and Coreboot source)

# Updating/Installing Dependencies

Before we can start, we will update our system to ensure that we have the latest packages. Let's open our system Shell and run the following:

```
sudo apt update
```
```
sudo apt upgrade
```

Let's install the packages we will need to read/write to our internal chipset:


```
sudo apt install flashrom
```
```
sudo apt install coreboot-utils
```
```
sudo apt-get install -y bison build-essential curl flex git gnat libncurses-dev libssl-dev zlib1g-dev pkgconf
```

FlashROM may already be installed on your system by default, but this just helps us to confirm that it is on our system and installed in full. "coreboot-utils" contains FlashROM utilities and a couple of other useful packages we will need.

*At this time, we should now have all of the necessary packages. Restart your computer.*

# ⚠️ DANGER! AN IMPORTANT NOTE: ⚠️

Before we do any writing operations to our chip, we need to make some critical backups so that we can restore our machine in the event of a ***flash failure***. A *flash failure* occurs if we make an error when we write our Coreboot image to our chip, which may cause it to boot incorrectly. This may result in our system be completely bricked.

We can reduce the risk of bricking our MacBook by backing up the ***Factory Firmware*** to our USB storage device. (So we can get the firmware backup if the computer is compromised)

Put simply, if you brick your MacBook, you may not be able to operate the keyboard and screen to reflash and recover your chip. If this happens, the only way to recover your MacBook is to ***EXTERNALLY FLASH*** it with a second computer. 

This would involve attaching clamps directly to your MacBook's motherboard and flashing your firmware backup directly to your chip from the second computer.

***I will write an external flashing guide in the future if enough people request it.***

Unfortunately, this is easier said than done. Some MacBook models support external flashing, but some do not. You can find a list of supported MacBooks [***HERE.***](https://ch1p.io/coreboot-macbook-support/)


## ⚠️ Regardless of whether or not your MacBook supports ***EXTERNAL FLASHING*** or not, treat this procedure as if you only have ***1 shot*** and ***NO DO-OVERS.*** ⚠️

# STEP 1: Backing Up Our Factory Firmware

Lets start by dumping our entire chip to a .bin file to serve as a backup. Insert your USB storage device into your computer. We will be backing up our chip and editing our firmware files on it so that it stays seperate from our host machine. 

Create a folder on your chosen USB drive and name it something short. I am naming mine "CorebootBackup". Inside, create a .bin file, and name it something easy to remember. I am calling mine "FirmwareOriginalBackup.bin".

Now, we will save a backup dump of our factory firmware to the empty .bin file:

```
sudo flashrom -p internal -c MX25L6405 -r /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin
```

Our chip's flash data has now been dumped and saved to the .bin we created.


Using "ifdtool", which we installed earlier through "coreboot-utils", we will write the layout of our backup to a text file so that we can see the layout of our chip's firmware regions.

```
sudo ifdtool -f FirmwareOriginalBackupLayout.txt /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin
```

Our flash layout should now be in our newly-created text file. Keep in mind, unless you specified the directory, it will have been created in your /home directory. 

Let's check what's inside:

```
cat FirmwareOriginalBackupLayout.txt
```

It should display something like this:

```
Line 1: 00000000:00000fff fd
Line 2: 00181000:007fffff bios
Line 3: 00001000:00180fff me
```

Using ifdtool, we will extract these flash regions from our original .bin dump that we saved as a backup earlier.

```
ifdtool -x /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin
```

This command should have extracted each flash region to some newly-created files on our system. The names should be as follows:

```
flashregion_0_flashdescriptor.bin
flashregion_1_bios.bin
flashregion_2_intel_me.bin
```

Again, they should have been created in our /home. Let's move them to our backup USB for safe-keeping.


# STEP 2: Gutting Intel ME Region

If you check the size of "flashregion_2_intel_me.bin", you will see that it contains 1.5 Megabytes of data. Using a Python script called "me_cleaner", written by Nicola Corna, ("corna" on GitHub) we can remove as much data from this file as possible and shrink it to by only Kilobytes in size.

After a quick Google, we can find the me_cleaner page on GitHub, which we can download the script from. After downloading the repository as a .ZIP, unzip it in the directory you want to work in. I put mine on the same USB where I am saving my backup files for organization purposes.

Now that we have downloaded me_cleaner, we can run it in the Shell to truncate "flashregion_2_intel_me.bin" and shrink it.

```
sudo python3 /media/sudo-judo/BACKUPDRIVE/me_cleaner.py -r -t -O flash_region_2_intel_me_truncated.bin /media/sudo-judo/BACKUPDRIVE/CorebootBackup/flashregion_2_intel_me.bin
```

A new file named "flashregion_2_intel_me_truncated.bin" should have been created in your home directory. This is a shrunken copy of your original. Cut and paste the file onto your backup USB. Let's create a new folder for our "modified" files called "CorebootNewFiles".


If we check the file's properties, we can see that it is only 81.9 Kilobytes, compared to the original 1.5 Megabytes. Success! The Intel ME region file has been truncated!

Let's copy and paste our Shell text output to our USB for safe-keeping.

# STEP 3: Creating A New flash Layout

In our new files folder, (remember, I made one called "CorebootNewFiles) I will create a new layout text file that we will use for our new addresses. I am going to name it "new_layout.txt" so we can keep track of it.

Now, let's write some new addresses to this text file as shown below:

```
LINE 1: 00000000:00000fff fd
LINE 2: 00001000:00020fff me
LINE 3: 00021000:000fffff bios
LINE 4: 00100000:007fffff pd
```

*These addresses are identical to the addresses gch1p uses for his Coreboot IFD hack, which you can find [HERE.](https://ch1p.io/coreboot-macbook-internal-flashing/)

Once done, save the text file.

Now, lets use ***ifdtool*** on the file to create a new modified version of our original .bin dump file.

```
sudo ifdtool -n /media/sudo-judo/BACKUPDRIVE/CorebootNewFiles/new_layout.txt /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin
```

A new file named ***"FirmwareOriginalBackup.bin.new"*** has now been created in the same directory as the original "FirmwareOriginalBackup.bin". Let's move it to our special folder ***"CorebootNewFiles".***

Let's extract the updated flash regions from ***"FirmwareOriginalBackup.bin.new".***

```
sudo ifdtool -x /media/sudo-judo/BACKUPDRIVE/CorebootNewFiles/FirmwareOriginalBackup.bin.new
```

4 files should have been created in your home directory. They should be named as follows:

```
LINE 1: flashregion_0_flashdescriptor.bin
LINE 2: flashregion_1_bios.bin
LINE 3: flashregion_2_intel_me.bin
LINE 4: flashregion_4_platform_data.bin
```

Let's move these files to our USB, in ***"CorebootNewFiles"***, our current workspace folder.

Phew! That was tedious, but if we did everything correctly, we should now have a copy of our original chip firmware stored in ***"CorebootBackup"***, and all of the modified copies we will use for our custom firmware image stored in "CorebootNewFiles", both of which are now safely on our USB.

Now that we have all of the necessary files, we can now flash Coreboot itself, alongside our custom files.

# STEP 4: Downloading Coreboot Source

We will need an internet connection for this next part. We will be downloading the files we need containing the Coreboot firmware. 

Visit the URL below to automatically download the archived branch:

```
https://review.coreboot.org/changes/coreboot~33151/revisions/24/archive?format=tgz
```

Extract your archive to your preferred directory. Don't move it to your storage USB just yet, as the build process may take longer if the read/write speed is impeded by the USB data bottleneck.

# STEP 5: Configuring A Temporary Bite-Sized Coreboot Image

In the Shell, switch your working directory to your newly-extracted coreboot folder.

```
cd /home/sudo-judo/YourCorebootFolder
```

Let's build the toolchain we need for x64/x86 chips architectures:

```
make crossgcc-i386
```
Now, we will remove the default mainboard preset: (It is set to QEMU emulation by default, so we need to clear that before we can switch to an Apple motherboard)

```
make distclean
```

Now for the hard part, where we build our Coreboot image with all of the files we created previously. Use arrow keys to scroll, and the **"Y"** or **"N"** keys to select or deselect options.

**Where [*] is specified, the setting should be set to "Y"**

**Where [ ] is specified, the setting should be set to "N"**

⚠️ All relevant data must be inputted exactly as described below: ⚠️

```
make menuconfig
```

**Mainboard** --> 

       Mainboard Vendor --> Apple
       Mainboard Model --> Your MacBook Model Number
       ROM Chip Size --> Size of the ROM Chip, measured in Kilobytes. We will set it to 1024 Kilobytes (1 Megabyte)
       Size of CBFS filesystem in ROM --> Set value to "0x1000"

**Chipset** -->

       [*] Add Intel descriptor.bin file --> Set to "YES" and add the file path to your modified Intel Descriptor File
       [*]   Add Intel ME/TXE firmware --> Set to "YES" and add the file path to your truncated Intel ME file.
       [ ]     Verify the integrity of the supplied ME/TXE firmware --> Set to "NO".
       [ ]     Strip down the Intel ME/TXE firmware --> Set to "NO".
       [ ]  Protect flash regions --> Unlock Flash Regions

**Payload** --> 

       Payload to add --> Select your payload of choice. Keep in mind that SeaBIOS may not be operatable with the internal MacBook keyboard.

Save and exit. You should be sent back to the Shell. Let's assemble our config file:

```
make
```

Our 1024 Kilobyte-sized Coreboot image should be built and located in /build in our Coreboot directory.





# Work In Progress...

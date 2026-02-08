Last Updated: 2/7/26


First, let's open a new system Shell window.

Before we can start, we will update our system to ensure that it is up-to-date. 

>>> sudo apt update

>>> sudo apt upgrade

Let's install the packages we will need to read and write to our internal chipset.

>>> sudo apt install flashrom

>>> sudo apt install coreboot-utils

>>> sudo apt-get install -y bison build-essential curl flex git gnat libncurses-dev libssl-dev zlib1g-dev pkgconf


FlashROM may already be installed on your system by default, but this just helps us to confirm. "coreboot-utils" contains FlashROM utilities and a couple of other useful packages we will need.

At this time, we should have all of the necessary packages.

Now, we will POWER DOWN our computer, and turn it back on (we want a COLD BOOT, not a restart)


***BACKING UP OUR CHIP***

Before we do any writing operations to our chip, we need to make some critical backups so that we can restore our machine in the event of a flash failure. There is one caveat to this: I am currently using the MacBook A1278, which is very difficult to externally flash. If you brick this model, it is very difficult to recover.

Put simply, if you brick your MacBook, you obviously can't operate the keyboard and screen to do any recovery steps. Recovery would have to be done with a second computer, attached directly to the motherboard with a SOIC-8 chip clamp, which can damage a LOT of delicate components. Externally flashing by sending payloads from another computer is very risky on the A1278.

If you are on a computer that supports external flashing, it will be much easier to flash your backup back to the chip if you know what you are doing.

Lets start by dumping our entire chip to a .bin file to serve as a backup. Please save this on a USB or other storage device outside of your computer.


Create a folder on your chosen USB drive and name it something short. I am naming mine "CorebootBackup". Inside, create a .bin file, and name it something easy to remember. I am calling mine "FirmwareOriginalBackup.bin".


Now we will save a backup dump of our factory firmware to the empty .bin file.


>>> sudo flashrom -p internal -c MX25L6405 -r /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin



Our chip's flash data has now been dumped and saved to the .bin we created.


Using "ifdtool", which we installed earlier through "coreboot-utils", we will write the layout of our backup to a text file so that we can see the layout of our chip's firmware regions.


>>> sudo ifdtool -f FirmwareOriginalBackupLayout.txt /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin


Our flash layout should now be in our newly-created text file. Keep in mind, unless you specified the directory, it will have been created in your /home directory. 

Let's check what's inside:


>>> cat FirmwareOriginalBackupLayout.txt


It should display something like this:

       |-----------------------------------
Line 1 |00000000:00000fff fd
Line 2 |00181000:007fffff bios
Line 3 |00001000:00180fff me
       |-----------------------------------


Using ifdtool, we will extract these flash regions from our original .bin dump that we saved as a backup earlier.

>>> ifdtool -x /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin

This command should have extracted each flash region to some newly-created files on our system. The names should be as follows:


------------------------------------------
flashregion_0_flashdescriptor.bin
flashregion_1_bios.bin
flashregion_2_intel_me.bin
------------------------------------------


Again, they should have been created in our /home. Let's move them to our backup USB for safe-keeping.


***Gutting Intel ME***

If you check the size of "flashregion_2_intel_me.bin", you will see that it contains 1.5 Megabytes of data. Using a Python script called "me_cleaner", written by Nicola Corna, ("corna" on GitHub) we can remove as much data from this file as possible and shrink it to by only Kilobytes in size.

After a quick Google, we can find the me_cleaner page on GitHub, which we can download the script from. After downloading the repository as a .ZIP, unzip it in the directory you want to work in. I put mine on the same USB where I am saving my backup files for organization purposes.

Now that we have downloaded me_cleaner, we can run it in the Shell to truncate "flashregion_2_intel_me.bin" and shrink it.

>>> sudo python3 /media/sudo-judo/BACKUPDRIVE/me_cleaner.py -r -t -O flash_region_2_intel_me_truncated.bin /media/sudo-judo/BACKUPDRIVE/CorebootBackup/flashregion_2_intel_me.bin

A new file named "flashregion_2_intel_me_truncated.bin" should have been created in your home directory. This is a shrunken copy of your original. Cut and paste the file onto your backup USB. Let's create a new folder for our "modified" files called "CorebootNewFiles".


If we check the file's properties, we can see that it is only 81.9 Kilobytes, compared to the original 1.5 Megabytes. Success! We have surgically neutered our Intel ME file!

Let's copy and paste our Shell text output to our USB for safe-keeping.

***NEW FLASH LAYOUT AND MAKING NEW FILES***

In our new files folder, (remember, I made one called "CorebootNewFiles) I will create a new layout text file that we will use for our new addresses. I am going to name it "new_layout.txt" so we can keep track of it.

Now, let's write some new addresses to this text file as shown below:

       -------------------------------
LINE 1 |00000000:00000fff fd
LINE 2 |00001000:00020fff me
LINE 3 |00021000:000fffff bios
LINE 4 |00100000:007fffff pd
       -------------------------------

*These addresses are identical to the addresses gch1p uses for his Coreboot IFD hack, which you can find here: 

https://ch1p.io/coreboot-macbook-internal-flashing/

Once done, save the file.

Now, lets use ifdtool on the file to create a new modified version of our original .bin dump file.

>>> sudo ifdtool -n /media/sudo-judo/BACKUPDRIVE/CorebootNewFiles/new_layout.txt /media/sudo-judo/BACKUPDRIVE/CorebootBackup/FirmwareOriginalBackup.bin


A new file named "FirmwareOriginalBackup.bin.new" has now been created in the same directory as the original "FirmwareOriginalBackup.bin". Let's move it to our special folder "CorebootNewFiles".

Let's extract the updated flash regions from ""FirmwareOriginalBackup.bin.new".

>>> sudo ifdtool -x /media/sudo-judo/BACKUPDRIVE/CorebootNewFiles/FirmwareOriginalBackup.bin.new

4 files should have been created in your home directory. They should be named as follows:

------------------------------------------
flashregion_0_flashdescriptor.bin
flashregion_1_bios.bin
flashregion_2_intel_me.bin
flashregion_4_platform_data.bin
------------------------------------------

Let's move these files to our USB, in "CorebootNewFiles", our current workspace folder.

Phew! That was tedious, but if we did everything correctly, we should now have a copy of our original chip firmware stored in "CorebootBackup", and all of the modified copies we will use for our custom firmware image stored in "CorebootNewFiles", both of which are now safely on our USB.

Now that we have all of the necessary files, we can now flash Coreboot itself, alongside our custom files.

***Downloading Coreboot Source***

We will need an internet connection for this next part. We will be downloading the files we need containing the Coreboot firmware. 

We have two options for this step:

1. Download the rolling release (bleeding edge, most recent, directly from git) by running: 

>>> git clone https://review.coreboot.org/coreboot.git

2. Download a stable release archive by navigating the Coreboot.org website.

*I will be installing a stable release copy

To download a Stable Release version, we'll go to Coreboot.org and navigate to the "Downloads" page. Find the Coreboot version date you want to install and click the download icon under the "Source" column. Specify the directory you want to download it to.

@@@ I recommend saving it to your host machine for now. The .tar archive needs to be extracted and it can take a long time if it is saved on the USB. We will move it to our USB after extraction. @@@

Extract your .tar archive in the same directory that it is already in. Don't extract to your USB yet.




***Configuring Coreboot***

In the Shell, switch your working directory to your newly-extracted coreboot folder.

>>> cd /home/sudo-judo/Downloads/coreboot-25.12

>>> make i386-toolchain

>>> make menuconfig





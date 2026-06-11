# What is it?
This repo contains the files needed to create a very tiny barebones Linux distro that has 1 purpose. run the popular DOOM game on x86_64 efi hardware. This is one of those random brainspins that sometimes keeps me up at night/gets in my head when i least expect it.


# Design/questions
SO below some design/engineering decisions and stuff. 


### Is it really as small as it seems?
#### Kernel
Yes it is! For the kernel i used tinyconfig which basically only enables the absolute required stuff for the kernel binary to even be a kernel, but it does go pretty far to even disable tty or printk and it disabled graphics, ound, usb, hid, input, basically everything. Then through lots of testing and trial and error i got a kernel of 1.6 Megabytes that at minimum boots up and has basic framebuffer graphics and keyboard input, which is needed for fbdoom. This kernel works on most (U)EFI systems as most systems support a minimal EFI framebuffer (so no hardware acceleration or any of that junk). The kernel does not use any (external) kernel modules.

#### Rootfs
SO the rootfs makes use of barebones busybox. Here i used the 'allnoconfig' which again pretty much disables everything. Then i enabled only the core commands that make the system functional and working and i even included a tiny bit of extra "bloat" which made debugging easier and makes it so you have a minimal shell where you can print stuff in and you can read files using cat. But there is no 'ps' or any other commands. The commands that are available are: 'busybox', 'cat', 'echo', 'ls', 'mount', 'sh'. 'sleep', 'tar', 'poweroff', 'clear', 'free'. Thats all there is for functionality. It really is primarily focussed on running the game at boot and nothing else. You can shrink it down even more and remove the majority of those commands but i found this a perfect balance between useability and the 'minimal rootfs' challenge. This rootfs makes use of a static busybox binary so no /lib folder or external libraries are needed. 

#### In short:
The core folders are: /bin, /dev, /proc, /sbin, /sys, /usr. Then there is an 'init' script which is just a shell script and a DOOM.tar.xz archive containing the game files and a 'release' file with a single string inside with the version. 
SO in short: Kernel is 1.6 Megabytes and RAMDISK is 2.3 Megabytes.

### How does it work (boot process)?
So at boot the kernel and RAMDISK are loaded into memory. Then the kernel gets executed and passed to it is the doomfs.cpio.gz RAMDISK. This RAMDISK contains the super tiny filesystem with only the absolutely needed files. This RAMDISK gets decompressed into memory (RAM, hence its called RAMDISK) and then the kernel uses that as the root filesystem. Then it runs my init script which is a simple shell script that mounts the core directories (so /proc, /dev, /sys) and then uncompresses the main DOOM.tar.xz archive which contains the fbdoom binary, doom1.wad WAD file and a small boostrap script. When that is decompressed, it runs the bootstrap script which does some file checking, making sure everything is there. If that check pases, it launches the fbdoom binary which is the game. When you exit the game, you will land into a small busybox shell, where you have access to basic commands like: 'ls', 'free', 'cat', 'echo', 'mount', 'sh', 'sleep', 'tar', 'poweroff', 'clear'. I pretty much stripped everything out of this shell, so its not usefull for much other than printing some text in the terminal and reading files using cat or checking free memory but then again, its a purpoe built ultra small filesystem with only the core stuff and its primary task is running the game. If you exit this shell it will poweroff the system by running the poweroff binary. The RAMDISK makes it so this filesystem is temporary and its all loaded into ram so wont touch any files on your local hdd and it does not really have the functionality to do so. Also when you somehow break the filesytem, because its a static archive (doomfs.cpio.gz) that is unpacked into ram, everytime at boot, it means you only need to reboot it to restore any f ups. 

### Why is there a DOOM.tar.xz archive, why not just the bare files?
Because i found out that the fbdoom binary together with its WAD file and bootstrap script i made took up around 5.5 Megabytes and i wanted to shrink that down so we could get the <3 Megabyte filesystem. So i compressed it as a tar.xz archive to shrink it down to 1.8 Megabytes which is super tiny! Also i saw on some IP cam i was taking a look at the other day, that the vendors also packaged the core binary into a compressed archive to save space so i also thought 'why not?'. 

#### DId you use AI?
I did not use any AI in this project. Not only am i highly againts AI slop and vibecoding, in general this project is too easy/short to make without any AI usage needed so i did not use any AI model. Not even in the debugging/configuration phase.

### Limitations
So as i might have said earlier, there is no hardware accelerated graphics as everything makes use of the plain old framebuffer, so gameplay might be slugish/bad. Also there is no sound as there are no drivers enabled for it and the fbdoom binary i compiled has no sound. And some other drivers that might be needed are not enabled as this kernel is stripped to its bare core ofcourse. But my goal was not gameplay. It was rather a challenge to myself to make a super duper small linux distro that only has 1 task and thats to run DOOM and i succeeded at that! This is not even a serious project with any dedication and it was more a weird random brain spark that i came up with and is one of those million weird useless projects that are completely useless/dont benefit anything, but are fun to make just to get it out of my head. I also came up with and made it in 45 minutes (excluding any compilation time). 

# How to build?
1. Clone this repository using git.
2. cd into the directory 'cd LinDo-Main'
3. make the build_lindo.sh script executable by running this command in the LinDo-Main folder: 'chmod +x build_lindo.sh' 


# Conclusion/END
SO i am really happy how it turned out and find it really cool that i managed to make a super tiny rootfs using bare busybox absolutely stripped down to its bare core and that the rootfs fits in 2.3 Megabytes of space and the kernel is 1.6 Megabytes and that it on most hardware just runs most of the time. And its crazy that i managed to strip Linux and busybox to their bare components and it still somehow loads and boots up. It also is surprisingly fast booting, but thats because it does not have to worry about loading drivers/software and most of the delay is put in artificially to make it so things are atleast readable. Things my Init script says and the bootstrap script says.

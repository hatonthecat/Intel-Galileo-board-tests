# Intel-Galileo-board-tests
Testing an Intel Galileo board (2013-14) purchased on eBay.

https://inavoyage.blogspot.com/2026/06/testing-galileo-board-in-2026.html

The lights power on, but it seems the driver isn't being recognized yet due to a proper OS and/driver mismatch, or procedure not followed. It's possible the firmware is bricked but i dont have enough tools at the moment to test that yet.

https://inavoyage.blogspot.com/2026/01/because-they-didnt-know-what-to-do-with.html (video)

Additional resources:

https://blog.arduino.cc/2013/12/06/a-step-by-step-tutorial-for-intel-galileo/

https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/x86/RTOS_Intel_Quark_Galileo_GCC

https://github.com/janunezc/GALILEO/wiki/Setup-on-Linux-PC

11:37AM
--

I might have figured out the cause of one of my Arduino and Firmware and Serial to USB issues:

https://share.google/aimode/em9DJcqVsNSf9N7qp

See file "null pointer Java 32 bit solution found via Gemini.odt"

"The Exception in thread "serial-input" java.lang.NullPointerException error occurs because the Intel Galileo Flash Tool (or Arduino IDE) lost communication with the board's serial port during a firmware flash update.This Java exception means the software tried to read from or write to a COM/Serial port object that was suddenly disconnected, uninitialized, or blocked by the operating system.

Quick Fix 

Steps

1. Correct the Power and USB Cable SequenceThe Intel Galileo requires external power to stabilize its serial communication. Interrupted sequences trigger null pointers.Disconnect everything (both the USB cable and power adapter).Plug in the 5V power supply first. Always power the board before connecting it to your PC.Wait 10–15 seconds for the onboard Linux operating system to fully boot up.Plug in the Micro-USB cable to the USB Client port (nearest the Ethernet jack), not the USB Host port.

2. Install the Proper 32-bit Java Runtime (JRE)The legacy Intel Galileo Firmware Updater relies on 32-bit serial drivers that crash on modern 64-bit Java environments.Download and install a 32-bit (x86) version of the Java Development Kit (JDK) or JRE (Java 8 is highly recommended for compatibility).

3. Update or Re-assign the COM Port DriversWindows often fails to recognize or drops the Galileo driver during a flash, leaving the serial reference null.Open Device Manager on your computer.Look under Ports (COM & LPT).If you see "Gadget Serial v2.4" with a yellow exclamation mark, right-click it and select Update Driver.Point the update path manually to the hardware\tools\x86\drivers folder located inside your Galileo-compatible Arduino IDE directory.

4. Close Conflicting SoftwareOnly one application can use the serial port at a time.Ensure you don't have the Arduino Serial Monitor open while running the standalone Flash Tool.Close any background terminal software (such as PuTTY, Tera Term, or Cura) that might be scanning your COM ports.

5. Fallback:

Flash via a Blank MicroSD CardIf the Java application refuses to cooperate, you can bypass the flash tool altogether:Format a MicroSD card (32GB or smaller) to FAT32.Download the official "Galileo Capsule" firmware binary file (.cap).Rename the file to galileo_firmware.cap and place it on the root of the SD card.Insert the card into the powered-down Galileo, then power it on. The board will automatically update its firmware from the card over the course of a few minutes."

6-23-2026
--

Part 2 outline of what I'd like to accomplish: https://inavoyage.blogspot.com/2026/06/testing-galileo-board-in-2026-part-2-of.html

More succinctly, I am trying to install linux on the Galileo with a desktop environment, and then use SSH with X11 forwarding to "view" the OS from my PC (either via USB-Serial, which I have spent days having issues with the driver, at least on Windows tests, or via LAN, which I am now testing).

The LAN method at least has some conveniences- knowing it's actually connected the router, for example. I have established 192.168.1.2 as the new IP address (yesterday it was .3)

I have used various flashing software. I have linux and Windows on 2 separate machines (3 OSes on 3 drives, technically), and got an interesting progression today with Rawrite32 (yes, for Windows). The first time I tried flashing it said there was an error writing- I was using a MicroSD to SD adapter, which was plugged into an SD slot on my i7 3720QM laptop. The exact steps taken were this:

1. Used Raspberry Pi Imager to Erase 32GB SDHC disc.
2. Used Rawrite32 to write 269MB image (7z compressed) of Galileo.img_v.1.0.0.7z. Displayed one error #5 writing to disk, but otherwise said complete .

 <img width="543" height="224" alt="access" src="https://github.com/user-attachments/assets/ebd05e79-23c1-407b-9b57-6ab82828f20f" />

Tried to SSH into board. Conenction refused. Probably didn't write correctly.

Found a different image, this one labeled galileo-1.2.img and tried to write it. Also ran Wind Disk Imager as Admin. Also used a USB adapter instead of the in-laptop SDHC slot:

<img width="555" height="315" alt="Sucessfully written to SDHC using SDXC 3 0 reader for Galileo" src="https://github.com/user-attachments/assets/2d33c7f5-acc9-41fb-a087-5872c0e9b6c3" />

There were no errors in the writing process, but I had doubts as to whether it contained all the useful software needed to SSH. It just occurred to me that I should erase the disk with at least a Quick Format on Rpi Imager before overwriting it, in case old sectors were read by the Galileo firmware. The fix could have been any one of the changes that were made in the 2nd attempt. 

Also found an archived page of instructions on how to formally flash the SD card from Intel:
https://web.archive.org/web/20181010091008/https://software.intel.com/en-us/get-started-galileo-linux-step1

Interestingly, yesterday I recall reading a blog that mentioned the write speed on one config file was set to SDXC levels, so they lowered the speed and it flashed correctly, although I don't recall if it was specific to a Galileo linux image. That could have been why it flashed without errors on the SDXC adapter (might have been forced to a slower speed than what the SDHC slot might have attempted), although I have no way of checking without some tinkering. Of the many other previous flashes, it's possible that one of the other images I wrote did have a working SSH, but I hadn't remoted into it at the time when all I has was a USB-Serial adapter with a non working drive. I suppose I could test both connections/cables now with multiple images and additional image writers. 

There are a lot of brilliant developers who have tested this board successfully. I am so behind on familiarizing myself with this board. I am sure some of the working solutions and knowledge about this board has been forgotten, and it sometimes gets an incomplete restoration. Many hidden quirks have been ironed out by someone on the web, but no one has a complete monopoly on all the fixes and applications, fortunately...Still, that makes hunting for a straightforward install much harder.

A week ago, I started reading "Where Wizards Stay Up Late" (1996). The author emphasizes at several points that a lot of the BBN technologists were in the top 1% of the top 1% of their class, or amongst hackers, the most brilliant at fixing the rarest bugs, like a synchronization bug (Crowther). This board may not be as complicated as a sync timer, despite having one(?), and some of that objectivity in standardized testing might be viewed a bit differently today, but it is remarkable how much knowledge is required to build it, and yet seemingly few actually did a thorough benchmark of all it's features/capabilities. It would be nice if I can scrape the surface of any of its functionality, and extraordinary if I didn't encounter a single bug.

2:45pm
---

In a strange twist of events, plugging in a microUSB cable to the Client-USB port on the Galileo board and the other end in a USB port on my laptop (right side) allowed Windows 10 to detect the COM3 driver, which Arduino 1.5.3 (the only Arduino version designed specifically for Galileo) now detects as the SAME COM3 port. I fortunately still had the firmware 1.1.10 and the Updater tool, so it seemed like a fresh opportunity to update the firmware and test out the Blink LED example test in Arduino! And sure enough, it worked! What I did first was Update Driver in Device Manager, where I located the Gadget Serial driver after the COM3 port was detected and added to the COM list:

<img width="408" height="456" alt="serial" src="https://github.com/user-attachments/assets/dcca8d0e-135f-4742-b85a-ac3cefe0d8fa" />
<img width="1222" height="611" alt="gadget driver" src="https://github.com/user-attachments/assets/fe8964b0-cce4-46e4-beb2-b27908bc0e59" />

Loading the Firmware Updater Tool, it did confirm that an earlier version of the Firmware was installed- 1.0.4. I had never been able to flash it before, and it finally worked. It took about 5 minutes, so don't unplug the power or USB cable while it is flashing. Fortunately there were no power outages (Plugging it into a battery with an inverter might be a safer bet if there is unstable power). I didn't use FRAPS, but I loaded it in case it would let me record (no built in encoder)

<img width="1477" height="715" alt="actually flashing" src="https://github.com/user-attachments/assets/1851333d-eb46-4961-bc45-d5422b9ae602" />

<img width="1502" height="620" alt="targfirm" src="https://github.com/user-attachments/assets/6c4b2b70-b92e-43b3-88d0-7acba6969dac" />

So I guess this resolves a long standing question I had about the Arduino software. I assumed it wouldn't work right because it needed a 32-bit OS, such as Windows 7, 8 or 10 instead of the 64 bit versions. That is quite a relief, since I would have avoided the need to run it in a VM or on another partition.

As I mentioned earlier, the first test I ran was Blink LED (under the Example bar). This is the most useful test to confirm the USB or LAN is actually sending commands to the board, and not randomly blinking. Strangely after playing around with PuTTY and TeraTerm, it seems like my router lost signal with the board, and no longer displays an IP address nor connection. There is still an orange light on the LAN port (orange and yellow, same as before I recall correctly, which may just indicate 100Mbps instead of 1Gbps), but I may be wrong.

As the saying goes, "two steps forward, one step back." The setback is that the LAN doesn't seem to work, but the Serial USB is at least able to connect, and it's the first time anything was able to ~~connect~~communicate. So maybe that is considered two steps forward.

I already restarted the router, but I suppose one flaw I did in flashing was I left the Cat5 cable plugged in instead of allowing the USB to flash the Serial alone with the power. Wasn't needed, most likely (I wonder if somehow it updated via the LAN despite not actually working subsequently. The prior eBAY owner could have installed their own firmware, for all I know- and phoned home. Interestingly, on one screen it DID display firmware "704" or "754" or "708", which might have indicated a custom firmware, although I did not recognize if that is an internal codename for the earlier 1.0.04 (possibly?)

Since I got it to communicate, I ran a few more PuTTY tests over ethernet, but it seems like the LAN isn't working at the moment via all applications and the router is most likely accurately reporting the Ip addresses. So I have a few options now, one is to find a complementary suite of apps I can flash the SD card, to potentially restore functionality in the LAN, and if not, at least do all the installations from the COM3 port and serial. The only thing is, I may need to run some commands from Windows 10 (e.g. Serial) whereas I may need to switch to Linux if I want to run some things over Ethernet, if I can get it to work again. Possibly a minor issue and a multi prong approach can debug a lot faster than being stuck on one distro.

How the correct driver finally installed? It's possible Windows ran a few updates in the background over the past few weeks and finally determined the right driver for my Galileo board. It's unclear why it wasn't able to find it sooner, but I doubt it was dependent on other updates that had to be installed. Maybe it had to revert to an older update after it sent multiple errors to Microsoft. In any case, I found that to be quite rare- a distro this recent wouldn't be guaranteed to support Galileo, which is over 12 years old. And Windows 10 hasn't gotten security updates in nearly a year (although unofficial updates have occasionally appeared in the Update queue.)

9:13 PM 
--
On my last "sabbatical" a few years ago, when I was studying Raku and PERL language, I had downloaded a bunch of Code Editors like VS Code, Comma, Vim variants, etcetera and I had preferred the Github Web editor for the most part, but I had installed Comma on this laptop around that time- the version I have installed is Comma 2022.10.  
I like the split screen and realtime preview- while it most certainly isn't the only one, the only thing I want to "fix" on this is to allow the sentences to automatically word wrap, even though this editor is designed for code and not word processing/publishing formats.
Maybe there is a way it can automatically generate a new line of "code" because the longer lines slow down the computer, since it's probably trying to parse/analyze it.

Writing in a code editor does create the constraint to want to write like Hemingway. To avoid lack word wrap features.

My original update was intended to cover a detail I read earlier today, where someone in a blog on Galileo tests years ago mentioned that the Galileo doesn't enable LAN by default. The fact that my router assigned an IP address suggests that one of the earlier firmwares or images I had in the SD card enabled the LAN at some point. 
It's possible the new firmware disabled it, although I found an Arduino sketch that might reenable it. 
Interestingly, I found that I when I tried to log into my Netgear router from one of my wifi PCs, it actually revealed that a new device from 192.168.1.23 is managing the device. 
I have seen this message only a couple times yesterday when I was logged in from one of my PCs, but never from the Galileo board. 
I do recall I enabled internet sharing with it yesterday, so it's possible that setting got transferred over.
I do hope it wasn't some SSH feature that got enabled from my PC by accident, allowing some rogue firmware/image to manage my entire home network (half joke). 
But I suppose it's possible since I am not a DEFCON cybersecurity expert.

The progress I am making is incremental, but I think it is getting a lot closer to running some actual linux software on the board. I do have both the minimal and larger linux images designed for Galileo, but I will have to review my notes from 3 weeks ago when I started these tests or try one of the Yocto project images hosted somewhere (some on github). If I can set up significant linux commands and point it to a maintained repository of dependencies, then I might be able download any remaining software on it once it is installed. Most likely, I will have to use a fairly old version of Yocto if I want to spin a new one, but fortunately I have enough RAM to compile in a short time (and I plan to compile multiple once I get good at it)

6/24/2026
--

10:30am

After watching a few tutorials on YT, I was surprised and totally forgot that Windows 9x systems depended on, more or less FAT16 until the mid 90s and is a requirement for writing the image to the SDHC card (At least one Galileo tutorial used FAT16 over FAT32, but I had to remind myself that image writing software doesn't automatically include disk formatting software, although the RPi imager includes a FAT32 erase. In this case, I used Windows CMD tool diskpart and instructables says that the FAT option doesn't appear if the driver is larger than 4GB in the GUI tool, but you can format the size first, or just use a software that directly formats to FAT16- MiniTool works alright too: https://www.instructables.com/Format-USB-Flash-Drive-to-FATFAT16-not-FAT32/. 

<img width="470" height="549" alt="image" src="https://github.com/user-attachments/assets/d1dd2e68-daef-472f-90fb-25484ae33ad3" />

Since I had the privilege of using HDDs in the mid 90s, I warmly recall using FDISK (to format drives ranging from 1GB to more than 4GB. The MSDOS menu would kindly make you aware that there was a 2GB partition size limit for the standard FAT16 using 32KB clusters. So, the maximum size some of these images are going to use is 2GB at most, even with a newer linux. I sometimes forget or get mixed up the differences between 16 bit operating systems  and filesystems, as they are not all the same, and probably can use the two on the same system. But... I will explore more that later. Right now, I need to use the standard FAT formatting, which Windows still conveniently offers in the GUI tool. 2GB is more than 2X my first HDD, so I don't think that will be limiting in any way. I would love to get ELKS Linux working on a 256MB filesystem, but maybe another day.

As Bill Gates once said*, "2GB ought to be enough storage for anyone"

*NOT.

I am curious of the speed of FAT16/B though. On a modern SDHC card, maybe there are some benefits? Or no. I have run into the 4GB limit on my Panasonic cameras before, so I know there is a nicety(?) to exFAT that the filesystem required . But I don't need that now.

5:44pm
--

Holy molex, I actually booted into linux on my Galileo board! I didn't think I would be able to get this far after just one day of getting it to blink LED (if you don't count the tests I did three weeks ago, but I have used some very composite methods, following a few tips from various tutorials, since I didn't exactly trust what they all had to say. Funnily enough, one tutorial said the Galileo supports NTFS and FAT32, whereas most places say it supports FAT and FAT32. As I used the diskpart to format to FAT, i left their options alone. After that I had a 2GB partition (the rest unallocated). Then I used Win32diskimager to write an unpacked bz2 (232MB->1.3GB) to the drive using [this tutorial](http://arduino.vn/bai-viet/525-intel-galileo-install-linux-yocto-iot-devkit-intel-galileo#:~:text=Galileo%20will%20automatically%20boot%20the,(leave%20blank%20and%20hit%20Enter).) (ignoring the NTFS & FAT32 advice) and managing to find an Intel server that still hosted the image: https://iotdk.intel.com/images/1.5/

Why I like this one: It includes "connman - is a command-line network manager designed for use with embedded devices and fast resolve times. It is modular through a plugin architecture, but has native dhcp and ntp support."

If you recall from earlier in my Part 2 of X blog post yesterday, I lost the IP address after I flashed the firmware, and while I could set up a direct Ethernet cable to my laptop, I preferred to leave that microUSB -Serial cable for other communications, such as Arduino sketches. 

After connecting to my Netgear intranet from my PC, initially it assigned an address in the 192.xxx. above 168.x, but then it received a fresh IP address and instead of saying <unknown> it actually had a name- GALILEO. So I loaded PuTTY, typed in the IP address, left SSH at the default radio dial with port 22 unchanged, and connected. It mentioned an unknown hash key, but since I trusted it - being on my intranet, I pressed yes and I was able to get a login prompt. It's important to use PuTTY and not CMD, including with administrative access, since CMD w/admin will only get you to a password prompt (and it's not clear which password it's asking for- I only tried the galileo one- which is empty and it said incorrect. With PuTTY it requests the username (root) and then the password should be empty.

Interestingly the installation may have disrupted some of the COM settings or drivers for Arduino and disappeared from the Device Manager. It's possible the linux image updated the firmware- I saw what appeared to be an older number, but most likely the frequent handling of the microUSB cable and unplugging it while getting the ethernet recognized caused it to temporarily disappear (to confirm, I wasn't able to send a blink LED from Arduino 1.5.3, as it only detected COM5 (a different COM), but I am not too concerned about that right now- I am more pleased with getting linux to boot, and to prove it, I am inclluding a picture of top running (redis, systemd, and other programs running- using nearly 120MB of RAM, more than half the system's ram when you count cached and reserved memory... There is also a video on youtube. What I would like to do next is get a bootloader prompt whenever the system boots up, with possibly multiple distros/live images, but that is not a huge priority- it might be one way of making use of <img width="1190" height="1284" alt="FIRST BOOT galileo" src="https://github.com/user-attachments/assets/775a0ed2-ba2b-498f-a14e-e4bf2d7d677a" />
larger microSD cards, which are the norm nowadays. Finding a 4GB or 2GB microSD isn't necessarily going to be a good use of one's money, unless they are bargain basement prices..

Anyways, I would like to move on to bigger things, like running Debian, or something like this one but with more features. I also have no idea if it can connect to the internet- getting the internet sharing enabled may be tricky, even if it is technically on the net- I don't know how to check. Also, the only way I can remote into the device is on my intranet, which isn't connected to the internet, since I am using a mobile hotspot, and even if I could share the device, I might need it to connect to the router via the USB port on the back, so that the PC can wire into the netgear, while getting internet over the wifi...long story.


<img width="1190" height="1284" alt="FIRST BOOT galileo" src="https://github.com/user-attachments/assets/0e9e8438-633c-4c9b-89db-cb8c126dae47" />

^The black line is actually from MS Paint. I know it looks amateurish, but I have a very good reason and I don't have an earlier copy so I will leave this image here. If you notice, I used the eraser to airbrush the MAC address in the netgear settings, which is why it's not visible. But the reason there's a black line is because the contrast on one of my monitors was set to very dark (it looks nicer for Youtube and videos), because I am trying to reduce eye strain, and had I left the brightness up, which I now see on another monitor, the black line would have been undone. No biggie. Crayon time is over.

The above image is what I saw before I used PuTTY for this SSH attempt. I do not recommend using CMD with admin to SSH into the board (as I did in an attempt after this because it doesn't ask for the username on the linux machine and it doesn't accept the empty password, because I don't know what I'm doing), so just use PuTTY or whichever SSH utility your machine has (TeraTerm also works on Windows):

<img width="992" height="1366" alt="DHCP" src="https://github.com/user-attachments/assets/f47d533e-8653-43b5-919c-ef79ad7cf9ed" />

https://www.youtube.com/watch?v=OqROsZosHsw&list=PLKvMTg3KKwP2c4D3414ua1VD9FkHAKGeO <-- There are 2 new videos in this playlist for Galileo- one is around 5 minutes long but you can ignore the end of it because I had accidentally set my Radeon recorder at a lower resolution, so when I resize it, the text is too blurry, hence I recorded a 2nd video with higher 4k resolution (even though it is just a region of the screen). You may have to sort the playlist by date because it will be at the bottom of the 8 videos (the first 6 are Yocto bitbake compilings on Ubuntu machines from months ago- you can ignore those, but I may actually bake a new one with all the software I want on the Galileo- all the shrimp you can eat. -Bubba Gump

7:17pm
--

Found a library that fixes the Quark X1000 segfault error, which allows Debian distros to avoid this particular error: https://github.com/assa77/libx1000
Apparently the vanilla Yocto images provided by the OE/Intel projects avoid this particular bug, hinting at awareness, but running Debian or another distro with more software packages/access is far more ideal. Although I am pretty sure the distro I want to built may benefit from a Yocto baking technique, over buildroot. In any case, this link is more of a mental note/reminder for future development.

I still plan to set up X11 forwarding and all that jazz, although I am unsure if I want too preinstall a window manager or allow it to download afer a minimal Debian is installed. I don't foresee any particular issues, unless installing a package on the Galileo runs into a problem during compiling (which would be on another machine). Anyways, I don't really need a lot of the extra packages, like node.js or even the perl/python compilers. I mean, I might need them, but I could download them later.

I guess I could figure out if the preinstalled opkg still points to something or I can route it somewhere else..first stop: X11?




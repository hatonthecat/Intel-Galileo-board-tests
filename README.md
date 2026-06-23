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

I have used various flashing software. I have linux and Windows on 2 separate machines (3 OSes on 2 drives, technically), and got an interesting progression today with Rawrite32 (yes, for Windows). The first time I tried flashing it said there was an error writing- I was using a MicroSD to SD adapter, which was plugged into an SD slot on my i7 3720WM laptop. The exact steps taken were this:

1. Used Raspberry Pi Imager to Erase 32GB SDHC disc.
2. Used Rawrite32 to write 269MB image (7z compressed) of Galileo.img_v.1.0.0.7z. Displayed one error #5 writing to disk, but otherwise said complete .

 <img width="543" height="224" alt="access" src="https://github.com/user-attachments/assets/ebd05e79-23c1-407b-9b57-6ab82828f20f" />

Tried to SSH into board. Conenction refused. Probably didn't write correctly.

Found a different image, this one labeled galileo-1.2.img and tried to write it. Also used a USB adapter instead of the in-laptop SDHC slot:

<img width="555" height="315" alt="Sucessfully written to SDHC using SDXC 3 0 reader for Galileo" src="https://github.com/user-attachments/assets/2d33c7f5-acc9-41fb-a087-5872c0e9b6c3" />

There were no errors in the writing process, but I had doubts as to whether it contained all the useful software needed to SSH. It just occurred to me that I should erase the disk with at least a Quick Format on Rpi Imager before overwriting it, in case old sectors were read by the Galileo firmware.

Also found an archived page of instructions on how to formally flash the SD card from Intel:
https://web.archive.org/web/20181010091008/https://software.intel.com/en-us/get-started-galileo-linux-step1

Interestingly, yesterday I recall reading a blog that mentioned the write speed on one config file was set to SDXC levels, so they lowered the speed and it flashed correctly, although I don't recall if it was specific to a Galileo linux image. That could have been why it flashed without errors on the SDXC adapter (might have been forced to a slower speed than what the SDHC slot might have attempted), although I have no way of checking without some tinkering. Of the many other previous flashes, it's possible that one of the other images I wrote did have a working SSH, but I hadn't remoted into it at the time when all I has was a USB-Serial adapter with a non working drive. I suppose I could test both connections/cables now with multiple images and additional image writers. 

There are a lot of brilliant developers who have tested this board successfully. I am so behind on familiarizing myself with this board. I am sure some of the working solutions and knowledge about this board has been forgotten, and it sometimes gets an incomplete restoration. Many hidden quirks have been ironed out by someone on the web, but no one has a complete monopoly on all the fixes and applications, fortunately...Still, that makes hunting for a straightforward install much harder.

A week ago, I started reading "Where Wizards Stay Up Late" (1996). The author emphasizes at several points that a lot of the BBN technologists were in the top 1% of the top 1% of their class, or amongst hackers, the most brilliant at fixing the rarest bugs, like a synchronization bug (Crowther). This board may not be as complicated as a sync timer, despite having one(?), and some of that objectivity in standardized testing might be viewed a bit differently today, but it is remarkable how much knowledge is required to build it, and yet seemingly few actually did a thorough benchmark of all it's features/capabilities. It would be nice if I can scrape the surface of any of its functionality, and extraordinary if I didn't encounter a single bug.

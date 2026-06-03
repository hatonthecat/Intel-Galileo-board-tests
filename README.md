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

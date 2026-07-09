the UART on the Galileo is 115,000 bps.

while one can use Ethernet to run Xserver over SSH/fbvnc, i was curious of the maximum tty/usb speed and how much would be needed for a small x window at 1-4 bits. https://share.google/aimode/wbFCTsjytucqSBbHz

as i have done some xwoaf tests in 4 bit, the frame buffer can be very small. but if the uart is only 115kbps, i needed to know if a small window could fit in that using 4 bit color or less. here is what Gemini has to say:

To calculate the exact theoretical bitrate for a raw 640x480 (VGA) video stream in 1-bit color, we use the formula: Width × Height × Bit Depth × Framerate. At 1 bit (monochrome/black and white) and 60 fps, the raw bitrate reaches exactly 18.432 Mbps, which is rarely used in standard streaming.Calculating Raw BitrateA 1-bit video simply means each pixel only uses 1 bit of data (which yields only two colors: black and white).Resolution: 640 × 480 = 307,200 pixels per frameBits per pixel: 1Total bits per frame: 307,200 bits (about 38.4 KB)To find the required bandwidth (bitrate) for uncompressed data, we multiply the bits per frame by the framerate (FPS):At 24 FPS: 7.37 MbpsAt 30 FPS: 9.21 MbpsAt 60 FPS: 18.43 MbpsAt 90 FPS: 27.65 MbpsStreaming & Compression ConsiderationsIf you are encoding this 640x480 video (using modern codecs like H.264 or H.265), the data rate drops drastically because codecs compress temporal redundancy (storing only the changes between frames rather than drawing every full frame).IP Security Camera standards: For standard 640x480 VGA streams in full color, manufacturers like Sony and Hikvision typically recommend bitrates between 256 kbps to 1 Mbps, depending heavily on the framerate and scene motion.1-bit specific use cases: Because a 1-bit image only contains binary data (black/white), encoders compress them incredibly well, meaning you can achieve great clarity at bitrates far below standard streaming recommendations (e.g., as low as 100 kbps for static content).To determine the exact bitrate you need for your specific hardware, refer to general FFmpeg Bitrate Guides or camera configuration manuals.

since the Galileo can't decode h.265/h.265, each frame would be 38KB, and 115kbps =14.4KBbps, so it would be actually 11kbps with overhead: 

"Standard 8N1 framing (8 data bits, 1 stop bit, no parity) introduces a 2-bit overhead per byte, meaning your true throughput will be around 11,520 bytes/s. For reliable communication at this speed, cables should be kept short (typically under a few meters), and your microcontroller should use an external crystal or PLL to avoid timing drift."

unless theres a way to only send the lines or quadrants of the screen, 38KB would consume more than three seconds of transfer rates. lowering the resolution is one option (320x240) = 9KB/s but then that"s only 1fps. additional scan lines could be sent sequentially and updated periodically.


https://forums.raspberrypi.com/viewtopic.php?t=32170

"Re: X forwarding via serial terminal (instead of SSH)
Thu Jan 16, 2014 10:44 am

I agree it would be one of those nice to haves. IMHO X-forwarding is simply genius.

Regarding bandwidth, it wouldn't be as bad as all that. I recall regularly using a x86 program called laplink which did do full remote desktop via serial - it wasn't fast...but it worked well enough for what I needed.

But yes I imagine a DIY solution would be a challenge to say the least. Wonder if the Wayland stuff has any tricks built-in to do this, since it uses its own alternative to X-Windows.

If reducing wires is an issue (but power is not - i.e. not running off battery), then a Wifi dongle could be a lot easier (with all the added bonus of full network link).

I guess for ease of use, you'd want the option to set it as an Access Point, so no router is needed (not tried that, but would be good to know how"

of course more typically one will get a "it can't be done" response: https://www.linuxquestions.org/questions/linux-software-2/running-xorg-on-a-serial-console-927982/ "When theres a will, theres a way"

https://en.wikipedia.org/wiki/Laplink

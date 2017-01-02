# Marlin 3D Printer Firmware for Anycubic I3
<img align="right" src="Documentation/Logo/Marlin%20Logo%20GitHub.png" />

 Marlin is the 3D Printer Firmware running on my Anycubic I3 printer. The stock firmware in Dec 2016 was based
 on Marlin 1.0.0 which was not able to compile with gcc 6.2.1 due to some added typedefs in the standard library,
 so I decided to update it to Marlin 1.0.2, the latest stable version as of Jan 2017. I compared the stock firmware
 to the original marlin source code and applied all changes that looked somewhat usefull to me and ignored those
 that didn't. **I cannot guarantee that my modified firmware will work. If you choose to install it, I am not
 responsible for whatever damages or other things might happen.**

## Compile it

To compile Marvin, open Arduino IDE, choose Arduino Mega 2560 as your board (I have a Ramps/1.4-compatible TRIGORILLA
motherboard on my printer) and compile the firmware. Make sure that there are no errors. Next, export the firmware
to a flashable .hex file. You probably want to use the one with bootloader.

## Flashing Marlin

Go and install yourself avrdude. I'd recommend using the latest development version, not the release. Don't forget
to add your user to the `uucp` group to have access to the printer when connected via USB. For ArchLinux users,
go and install `avrdude-svn` from the AUR.

The first thing you should do is to download the currently installed firmware, do this by executing

    avrdude -p m2560 -c stk500 -U flash:r:marlin-orig.hex:i -v

when the printer is connected via usb. You should have a file called `marlin-orig.hex` with the pre-installed firmware.
Keep it.

To flash Marlin, you could try to do it via USB as well:

    avrdude -p m2560 -c stk500 -U flash:w:Marlin.ino.with_bootloader.mega.hex -v

If this works for you, be happy. In my case it didn't work so I decided to use the ISP headers of my motherboard
and a RaspberryPi to upload the firmware. Install your Pi next to the printer, power it on and connect it to the
internet. Open an SSH shell and install avrdude. Put the firmware file on the Pi.

Next, we need to connect the two devices. The ISP header of the motherboard is numbered, the pins are like this:

![Arduino ISP header](https://www.arduino.cc/en/uploads/Tutorial/ISP.png)

Take 2 Male-Female-Jumper and connect them to GND and MISO. Connect another 3 Female-Female-Jumper to MOSI, SCK
and GND. Now, use 2 Resistors that are about 2:1 (I've used one 1.5k and one 2.7k) and wire it like this:

<img src="https://raw.githubusercontent.com/msrd0/MarlinAnycubicI3/anycubic-1.0.2-2/isp_bb.svg" type="text/svg" />

Now, open `/etc/avrdude.conf` on the Pi, find and uncomment section `linuxgpio` and change the values to this:

```
programmer
  id    = "linuxgpio";
  desc  = "Use the Linux sysfs interface to bitbang GPIO lines";
  type  = "linuxgpio";
  reset = 14;
  sck   = 11;
  mosi  = 10;
  miso  = 9;
;
```

Now, run

    avrdude -p m2560 -c linuxgpio -U flash:w:Marlin.ino.with_bootloader.mega.hex -v

and you are done!

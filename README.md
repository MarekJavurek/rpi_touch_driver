# changes made by me (Marek Javurek)

- reading not 25 but **22 bytes**!
- disabled multitouch, only one touch supported
- multiple output data by Calibration constants: -1843476 13679 16 -1707140 -48 8414 65536 (inspiration: https://github.com/saper-2/rpi-5inch-hdmi-touchscreen-driver)
- tested with Raspberry Pi Zero W (Raspbian) and 52Pi Free Driver 5 inch 800 * 480 TFT LCD HDMI Touch Screen Display (https://www.aliexpress.com/item/32314250806.html?spm=a2g0s.9042311.0.0.27424c4d7DnKYA)

## and just FYI, to have correct HDMI output you have to set in your confix.txt this:

```
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=87
hdmi_cvt=800 480 60 6 0 0 0
hdmi_drive=1
max_usb_current=1
```

# Original readme:

User-mode driver for touch part for:

7" inch Capacitive Touch Screen HDMI TFT LCD High Speed fr Raspberry Pi/B/B+/Pi2

A number of LCD displays with HDMI interface and capacitive touch
can be found on ebay and similar places.  When plugging in the usb cable,
Linux reports:

- usb 6-2: New USB device found, idVendor=0eef, idProduct=0005
- usb 6-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
- usb 6-2: Product: By ZH851
- usb 6-2: Manufacturer: RPI_TOUCH

The manufacturer (WaveShare?) is breaking every rule:
  - idVendor=0eef is assigned to D-WAV Scientific Co., Ltd, but the device is
    probably made by someone else.
  - totally bogus HID report descriptor
  - 16-bit data transferred in big-endian mode
  - GPL violation by distributing modified binary-only kernel (for RPi) where
    usbtouchscreen.c has been modified to support the first touch

Examples: 
 - http://www.waveshare.com/product/mini-pc/raspberry-pi/expansions/7inch-hdmi-lcd-b.htm
 - http://www.eleduino.com/7-0-inch-Hdmi-touch-with-USB-touch-Display-Support-Raspberry-pi-Bnana-PI-Banana-Pro-Beaglebone-bone-p10442.html

Touch events consists of 25 bytes, one example is

aa 01 03 1b 01 d2 bb 03 01 68 02 cc 00 5d 01 ef 01 5f 01 fe 00 fb 02 37 cc

Offset:
-     0 : Start byte (aa)
-     1 : Any touch (0=off,1=on)
-   2-3 : First touch X
-   4-5 : First touch Y
-     6 : Multi-touch start (bb)
-     7 : Bitmask for all touches (bit 0-4 (first-fifth), 0=off, 1=on)
-   8-9 : Second touch X
- 10-11 : Second touch Y
- 12-13 : Third touch X
- 14-15 : Third touch Y
- 16-17 : Fourth touch X
- 18-19 : Fourth touch Y
- 20-21 : Fifth touch X
- 22-23 : Fifth touch Y
-    24 : End byte (cc or 00)

This user mode driver decodes the touch events and injects them back into the
kernel using uinput.  The drives requires a kernel with uinput, and there is
nothing RPi-specific about it.

To use:
* % make && sudo make install
* sudo make systemd-install (only if your system uses systemd, otherwise you must find your own way to start the daemon)
* (not tested) If calibration is necessary, install 99-rpi-touch.conf to /etc/X11/xorg.conf.d,
calibrate using whatever calibration program you fancy, uncomment the calibration option line, and replace the calibration constants in 99-rpi-touch.conf


Copyright (c) 2015 Bjarne Steinsbo

Code and inspiration from http://thiemonge.org/getting-started-with-uinput
and the CyanogenMod userspace touchscreen driver for cypress ctma395.

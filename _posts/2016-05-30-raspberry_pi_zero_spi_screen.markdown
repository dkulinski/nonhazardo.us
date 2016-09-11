---
layout: post
title: Connecting a cheap Aliexpress TFT to the Raspberry Pi Zero using device tree
date: 2016-05-30 06:56:00
categories: raspberry pi tft spi device tree
---
##Overview
I recently acquired a Raspberry Pi Zero and a cheap 2.2 inch TFT LCD from Aliexpress.  The TFT is driven by a Ilitek ILI9341 chip in SPI mode.  SPI requires 4 wires for data transfer and the TFT requires a few extra connections for control. This should work on any Raspberry Pi with a 40 pin header (B+, 2 B, 3 B and Zero). This may also work on the original A and B but I have not tested this.

##Connecting the LCD to the Raspberry Pi
http://elinux.org has a great piece of documentation on the GPIO pins and their assignments.  Please reference it [here](http://elinux.org/RPi\_Low-level\_peripherals).

**UPDATE 08/21/16**
I mistakenly listed connecting the LED directly to a GPIO pin.  This will draw too much current and can potentially damage a Raspberry Pi board. There are two ways to overcome this flaw, either put a transistor in place with a connect to the 3.3v rail or just connect it directly to the 3.3v rail with an appropriate sized current limiting resistor.  

**UPDATE 09/09/16**
After struggling with a second Raspberry Pi Zero and one of these cheap 2.2 inch screens I have discovered that if you hook the Vcc to the 3v3 line then you need to solder the J1 jumper on the back of the board.  If you don't you may only get a white screen.
 
Connect all the SPI pins on the TFT to the corresponding pins on the Raspberry Pi.  Connect reset to GPIO 23, dc to GPIO 25 and LED to GPIO 24. Connect Vcc to a 3.3V pin and Gnd to a ground pin. 

##Device tree compiler
In order to enable the screen you will need to install the following device tree file.  In order for the file to work the device tree compiler is needed.  It can be installed with:

```bash
sudo apt-get install device-tree-compiler
```

Once the compile is installed, create a file called tft22.dts and paste in the following code:

```
/dts-v1/;
/plugin/;

/ {
	compatible = "brcm,bcm2835", "brcm,bcm2708", "brcm,bcm2709";

	fragment@0 {
		target = <&spi0>;
		__overlay__ {
			status = "okay";

			spidev@0{
				status = "disabled";
			};

			spidev@1{
				status = "disabled";
			};
		};
	};

	fragment@1 {
		target = <&spi0>;
		__overlay__ {
			/* needed to avoid dtc warning */
			#address-cells = <1>;
			#size-cells = <0>;
		
			tft22: tft22@0 {
				compatible = "ilitek,ili9341";
				reg = <0>;
				pinctrl-names = "default";

				spi-max-frequency = <48000000>;
				rotate = <270>;
				fps = <30>;
				bgr;
				buswidth = <8>;

				reset-gpios = <&gpio 23 0>;
				dc-gpios = <&gpio 25 0>;
				led-gpios = <&gpio 24 0>;
				debug = <0>;
				
			};
		};
	};

	__overrides__ {
		speed = <&tft22>,"spi-max-frequency:0";
		rotate = <&tft22>,"rotate:0";
		fps = <&tft22>,"fps:0";
		debug = <&tft22>,"debug:0";
	};
};
```

**UPDATE 6/4/2016**
If you are using a recent version of Raspbian you will need to use the following command:

```bash
dtc -@ -O dtb -I dts -o tft22.dtbo tft22.dts
```

Compile with the following command:

```bash
dtc -@ -O dtb -I dts -i tft22.dts -o tft22.dtb 
```


Move the dtb file to /boot/overlays

Add the following to the bottom of /boot/config.txt

```
dtparams=spi=on
dtoverlay=tft22
```

Add the following to /etc/rc.local to copy the console from the current location to the new /dev/fb1.

```
conmapfb 1 1
```
An X session can be started on the screen by replacing /dev/fb0 in /usr/share/X11/xorg.conf.d/99-fbturbo.conf to /dev/fb1.

Reboot and enjoy your screen!

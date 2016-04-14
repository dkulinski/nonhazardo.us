---
layout: post
title: Connecting a cheap Aliexpress TFT to the Raspberry Pi Zero using device tree
date: 2016-04-14 14:13:00
categories: raspberry pi tft spi device tree
---
##Overview
I recently acquired a Raspberry Pi Zero and a cheap 2.2 inch TFT LCD from Aliexpress.  The TFT is driven by a Ilitek ILI9341 chip in SPI mode.  SPI requires 4 wires for data transfer and the TFT requires a few extra connections for control.  

##Connecting the LCD to the Raspberry Pi
http://elinux.org has a great piece of documentation on the GPIO pins and their assignments.  Please reference it [here][http://elinux.org/RPi\_Low-level\_peripherals].

Connect all the SPI pins on the TFT to the corresponding pins on the Raspberry Pi.  Connect reset to GPIO 24, dc to GPIO 23 and LED to GPIO 24. Connect Vcc to a 3.3V pin and Gnd to a ground pin. 

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

An X session can be started on the screen by modifying /dev/fb0 in /usr/share/X11/xorg.conf.d/99-fbturbo.conf to /dev/fb1.

Reboot and enjoy your screen!

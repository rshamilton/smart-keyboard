---
title: "Smart Keyboard"
github: "rshamilton/smart_keyboard"
description: "A keyboard running on an ESP32 with a variety of smart features."
created_at: "11/3/2026"
total_time: "1.5hr"
---

# Smart Keyboard Journal

## 11/3/26 - Setup and outline the project

I got KiCad running off the drive it was installed on and created my initial plan after doing a lot of research into which components could be manually soldered and which would work well for this project.

- Wireless mechanical keyboard built with an ESP32-S3-WROOM-1-N16R2 (fewer pins to manually solder).
- Uses MX hot-swap switches.
- LiPo battery charged over USB-C.
- 1.3-inch OLED display.
- One push-click encoder for interacting with the screen and controlling volume.
- Piezo buzzer for encoder feedback and smart home integration.
- I2S microphone breakout to make soldering the tiny components easier.
- I will set up footprints for per-key RGB lighting and use them depending on available funding.
- Connect to my computer using Bluetooth or USB-C.
- Consider making an ESP-NOW dongle for a faster, more reliable connection.
- Use Wi-Fi for configuration, updates, internet data, and AI answers displayed on the OLED from typed or spoken questions.
- Use the API provided by Hack Club to give the keyboard an AI model for answering simple questions quickly.
- Set up a working folder and installed the Keyswitch KiCad Library.

Total time: 1hr
---

**Next up:**

Divide the schematic into separate sheets and begin wiring them while figuring out where everything should go.

![Project image](https://cdn.hackclub.com/01a1a41d-9303-75e9-973b-22e9a3779f62/paste-1793754762953.png)

## 11/4/26 - Schematics

I started by creating new schematics for the different sections:
- Matrix
- Peripherals
- Power
- Processing

I then got right to work on the programming schematic by dropping in the ESP-32 chip. I wired power into it and added buttons for BOOT and RESET. I also added 3V3 through a resistor to the EN pin so it is pulled on and wired it to ground through a capacitor to slow startup and allow power time to stabilize.

I then added decoupling capacitors to the design to filter out noise.

I added a label to the EN wire so the net has a readable name in the PCB editor and for a test point later.

Next I got the USB data lines ready for the Power sheet, where the USB-C connector and ESD protection will go. I drew short wires off the D- and D+ pins and put bidirectional hierarchical labels (USB_D- and USB_D+) on the ends so they can connect to the Power sheet once it's drawn.

Last, I added a 3-pin UART debug header so I can still get logs from the chip with a USB-to-serial adapter if USB isn't working during bring-up. I connected TXD0 and RXD0 to it with net labels instead of long wires, with GND on pin 1. I left 3V3 off so the board and the adapter can't back-feed power into each other.

I ran ERC to check my work. Everything I wired is connected. The remaining errors are the unused GPIOs, +3V3 having no source yet, and the USB labels not having a match on the Power sheet yet.

Total time: 3hrs

**Next up:**

Start the Power sheet with the USB-C connector, its CC resistors and ESD protection, then connect USB_D-/USB_D+ between the Power and Processing sheets on the root sheet.

![image](https://cdn.hackclub.com/01a1b317-a600-7b01-a72f-41d3e7199112/paste-1794006033558.png)

![image](https://cdn.hackclub.com/01a1b316-31cf-7451-9fd2-3284e6aed350/paste-1794005937918.png)

![image](https://cdn.hackclub.com/01a1b316-7996-752f-8c43-843cc6b5a6f1/paste-1794005956263.png)

![image](https://cdn.hackclub.com/01a1b316-e049-7ae6-8d44-c97025cc9bbb/paste-1794005983049.png)



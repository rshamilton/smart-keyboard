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


## 11/6/26 - USB-C and ESD protection

I started the Power sheet with the USB-C port. I used KiCad's 16-pin USB 2.0 receptacle symbol and set its value to the HRO TYPE-C-31-M-12, which is the connector I plan to use since it's cheap and has pins I can drag-solder.

- I tied the two D- pins together and the two D+ pins together, since USB-C has a pair of each so the cable works either way up.
- I added a 5.1k resistor from each CC pin to GND so a USB-C charger or computer recognizes the keyboard as a device and turns on 5V.
- I connected the shield and GND pins to ground.
- I put no-connect flags on SBU1 and SBU2 because USB 2.0 doesn't use them.
- I put a VBUS power symbol on the 5V pin so the charger can pick it up later on this same sheet.

Next I added a USBLC6-2SC6 ESD protection chip between the connector and the ESP32 on D- and D+, to protect the chip from static when the cable gets plugged in. I wired the data lines from the connector into one side of it, connected its VBUS and GND pins, and put USB_D- and USB_D+ hierarchical labels on the protected side.

With hierarchical labels on both the Power and Processing sheets, I went to the root sheet and added matching sheet pins to the Power and Processing boxes, then wired them together. That connects the ESP32's USB pins to the port.

I added PWR_FLAGs to VBUS and GND to tell ERC that the USB connector is what supplies them. Then I ran ERC, and the only errors left are the GPIOs I haven't used yet and +3V3, which won't have a source until I add the regulator.

Total time: 3hrs

**Next up:**

Add the LiPo battery charger (MCP73831) and battery connector to the Power sheet, then the power path and power switch.

![image](https://cdn.hackclub.com/01a1b386-56f5-71b9-93d5-b4a316c7b020/paste-1794013287724.png)

![image](https://cdn.hackclub.com/01a1b386-7bde-7aa3-bf9f-4615c348ca4e/paste-1794013297272.png)

![image](https://cdn.hackclub.com/01a1b386-96ca-727a-af94-caf307472797/paste-1794013304313.png)

![image](https://cdn.hackclub.com/01a1b386-ac62-7fbb-a993-10d50d4f6ca1/paste-1794013309298.png)

## 11/7/26 - Battery charger and power path

I added the battery charger to the Power sheet. I used the MCP73831-2-OT, which is the 4.2V version that matches a LiPo cell.

- I powered it from VBUS and put a 4.7uF capacitor on VBUS next to it.
- I added a 2k resistor from PROG to GND, which sets the charge current to 500mA, about a quarter of the 2000mAh battery's capacity.
- I put a +BATT power symbol on its VBAT output with another 4.7uF capacitor to GND.
- I left STAT open for now. It goes to the ESP32 with the rest of the sensing.

For the battery I added a 2-pin connector with the value set to B2B-PH-K-S, the JST-PH header I plan to use. Pin 1 is +BATT and pin 2 is GND.

Then I added the power path so the keyboard runs from USB when it's plugged in and from the battery otherwise. VBUS goes through an SS14 diode to the output, and the battery goes through an AO3401A MOSFET. The MOSFET's gate goes to VBUS with a 100k pull-down to GND, so it disconnects the battery whenever a USB is plugged in.

I put a slide switch (MSK-12C02) after the power path so the battery still charges while the keyboard is switched off. I used the common pin and one side, put a no-connect flag on the unused pin, and added a VSYS label to the output for the regulator to connect to.

I ran ERC. Along with the usual unused GPIOs and +3V3, it now flags STAT and the VSYS label, which only has the switch on it so far.

Total time: 3.5hrs

**Next up:**

Add the 3.3V regulator on VSYS and more battery stuff.

![image](https://cdn.hackclub.com/01a1b6ef-6d25-7d13-9a1d-37ec7aa91f44/paste-1794070505921.png)
![image](https://cdn.hackclub.com/01a1b6f0-e7ec-71c7-94b8-84234e420b69/paste-1794070603075.png)
![image](https://cdn.hackclub.com/01a1b6f0-36c5-71c5-b033-e4cba0923f04/paste-1794070557552.png)
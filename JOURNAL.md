---
title: "Home Assistant Control Pad"
author: "Pegoku"
description: "An ESP32-S2 based Control pad for Home Assistant."
created_at: "2025-05-18"
total_time_spent: "54h"
---

# May 18th: Sketch and first prototype!
To start, I did a simple sketch of what I would like for the Pad to be, and what features it should have. I wanted to keep it simple, so I focused on the most important features that would make it useful for controlling Home Assistant devices.

Features I ended up with:

1* Mode/View toggle buttons

2* Adjustment knobs (pots or encoders)

3* Action buttons

4* Toggle switches (Not used)

5* USB-C Charge port

6* display info (OLED)


![alt text](assets/journal/image.png)

I also tested how to drive an I2C dispay with the an ESP32-C3 and use multiple Screens activated with buttons.
Used this as reference: [SSD1306](https://esphome.io/components/display/ssd1306)

![alt text](assets/journal/image-1.png)

**Total time spent: 3h**

# May 19th: Using potentiometers and dynamically updating the display
I wanted to use potentiometers to adjust values, so I started testing how to read them with the ESP32-C3. I found that using the ADC pins wasn't as easy as I thought, as I had to set `attenuation: 12db` to allow 0~3.3V range.

![alt text](assets/journal/image-2.png)

PS: The potentiometer later broke and replaced it with a smaller one

**Total time spent: 4h**

# May 20th: Adding action buttons and indicator bar.
Next I wanted to add action buttons so I could trigger or toggle items in Home Assistant. To test, I used it to turn on and off a light.

![alt text](assets/journal/image-3.png)

I also added coded an indicator bar to show the current value of the device.

![alt text](assets/journal/image-4.png)

PS: While coding the bar, I found a bug (may also be user error) but it seems like if use the same variable twice for the same screen item(eg. rectangle). The esp32 refuses to connect to wifi, it also doesn't respond to actions (eg. button presses)

**Total time spent: 3h**

# May 21st: Find limitations of ESP32-C3
I started wiring the remaining action buttons and potentiometers, when I realised that the ESP32-C3 has a limitation of 11 GPIO pins. 3 of which are ADC. This means I'm unable to use all the buttons, potentiometers and the OLED display at the same time. So when designing the PCB I'll have to switch to the ESP32-S2.

![alt text](assets/journal/image-5.png)

**Total time spent: 2h**

# May 22nd: Start designing the Schematic
I started designing the schematic for the PCB, using the ESP32-S2. I used the ESP32-S2-MINI-2-N4R2, PEC11R-4220F-S0024 (rotary encoders) and B3F-4055 (tactile switches).

![alt text](assets/journal/image-6.png)

**Total time spent: 2h**

# May 23nd: Finish Schematic and start PCB design
I selected the Buck-Boost (TPS631010YBGR) for both the 5V and 3.3V rails. I also added an USB-C port and Battery charging (MCP73833).

![alt text](assets/journal/image-7.png)

Then I added 18 neopixel LEDs to the PCB, which will only 4 ended up being used in the final design. the OLED display, Battery voltage sense and battery protection DWQ1A.

![alt text](assets/journal/image-8.png)

**Total time spent: 5h**

# May 24th: Continue PCB design and choose battery
I continued the PCB design, placing the components. I also added a battery to use, I wanted to use a 18650 but I thought it would be too big so I went with a [1800mAh 103450 LiPo](https://www.ebay.es/itm/255510046348) with integrated BMS. I also added a removed the battery protection circuit from the board as the battery already has one.

![alt text](assets/journal/image-9.png)

**Total time spent: 3h**

# May 25th: Finish PCB design! Also, start case design
I continued placing all the components, and finally finished the PCB design!

![alt text](assets/journal/image-10.png)

![alt text](assets/journal/image-11.png)

Once I finished the PCB design, I started designing the case in FreeCAD.

![alt text](assets/journal/image-12.png)

**Total time spent: 8h**

# June 1st: Finish case design!
I finished the case. Added buttons and a slot for the USB-C port. I also added holes for the buttons (which I also designed) and rotary encoders, also added a slot for the OLED display.

![alt text](assets/journal/image-13.png)

**Total time spent: 4h**

# July 4th: soldering the PCB
Today I soldered the PCBs, it was pretty straightforward, except for some tiny components like the buck-boost converter which were a bit tricky to solder, as I was unable to see the IC labels.
It was hard, but eventually I ended up with a fully soldered PCB!

![alt text](assets/build/PXL_20250704_202810597.MP.jpg)

Then, I tested the battery charging circuit, and it worked!!! I was able to charge the battery safely to 4.2V! (don't know how the leds work though, but idrc, it just works)

![alt text](assets/build/PXL_20250704_224240737.MP.jpg)

**Total time spent: 6h**

# August 2nd: 
Quite a long time has passed, but it was worth it! I went to the best hackathon ever, Undercity! It was an amazing experience, I met a lot of amazing people and learned a lot. I also got to build a cool project.
Continuing from where I left off. I started coding the firmware for the control pad. Luckily I had already done most of the work, as I already had a working version for the ESP32-C3, so I just had to adapt it to the ESP32-S2. I also added support for the rotary encoders. 
Sadly, the code wasn't working as expected. The rotary encoders registered, but they weren't showing on the display, so I debugged it for a while, I even thought it was a power issue, which in part it was, as I discovered the 5V boost wasn't behaving correctly. And that's why I tried wiring the display to 3.3V, with no luck.

![alt text](assets/build/PXL_20250802_150521126.MP.jpg)

Then, I debugged the code, and found that the issue was with the I2C bus. It seems that I wired the I2C pins incorrectly as no devices were being detected. 
I left it for the day, as I was tired and frustrated.
**Total time spent: 6h**

# August 3rd: Debugging the I2C bus
Today was getting close to the deadline to ship projects to highway, so I did a but of _multitasking_ and worked on this prject and the Smart desk lamp at the same time.
I still thought the problem with the display was a power issue, so I inspected the PCB and found that it had an 3.3V LDO, and I just bridged it so the display would get the direct 3.3V from the buck-boost converter. Sadly, it didn't help. I also tried with a diferent display, but it didn't work either. As a las resort, I configured the GPIOs 18 and 36 to be the I2C pins, they were for buttons, but I didn't need them for now. But again, that didn't work either, yes, I added pull-up resistors to the SDA and SCL lines, but it still didn't work. I was getting really frustrated, so I left it for the day.
I think I'll just not use the display for now. It'll just be able to control 4 devices, but I think It'll be enough, as I don't really have that many "smart" devices at home.

![alt text](assets/build/PXL_20250803_161150798.MP.jpg)

**Total time spent: 4h**

# August 5th: Final assembly
Today I assembled the case. Sadly not everything fit as I expected, so I had to do some modifications to the case. 
This are all the modifications I had to do:

![alt text](assets/build/PXL_20250805_183627054.MP.jpg) 

![alt text](assets/build/PXL_20250805_183645312.MP.jpg)

**Total time spent: 4h**
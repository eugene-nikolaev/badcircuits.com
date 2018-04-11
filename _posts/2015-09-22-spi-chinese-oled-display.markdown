---
layout: post
title:  Connecting non-standard Chinese OLED-display using SPI
date:   2015-09-22 09:59:03
categories: oled spi arduino
---
I want to share how to connect a non-standard Chinese OLED-display bought on ebay:

[0.96" I2C IIC SPI Serial 128X64 White OLED LCD LED Display Module for Arduino](https://rover.ebay.com/rover/1/711-53200-19255-0/1?icep_id=114&ipn=icep&toolid=20004&campid=5338194349&mpre=http%3A%2F%2Fwww.ebay.com%2Fitm%2F0-96-I2C-IIC-SPI-Serial-128X64-White-OLED-LCD-LED-Display-Module-for-Arduino-%2F263097814635)

It wasn't too easy because it didn't work with popular libraries which were in the most of instructions in web.
I chose it by only highlight color. That time I didn't know that they could be connected using different interfaces.

That's what I received: 

![Display Front View](/assets/01_oled_front.jpg)

Please note the pin labels. I saw a bit different pinout on some DIY websites. The number of pins might be different or they were labeled in other way.  
So I asked that ebay seller for a datasheet and I got a dropbox link very quickly. There I found a .rar archive file with a Chinese name. Though there was "OLED" among those hyeroglyphs.

There were another .rar archives inside it, like a russian doll. I saw usual UC names among hyeroglyphs — **MSP430**, **PIC**, **STM32**. At last, I found **Arduino**.
There were two archives for it — named **IIC** and **SPI**. What should I choose? I remembered there was something like this at the bottom of the display board:

![Display Bottom View](/assets/02_oled_back.jpg)


Looks like the display supports two interfaces. Which one is active could be selected by resistors;

```
4SPI: R3
IIC: R1, R6, R7, R8
```

**4SPI** — it is 4-wire SPI, this protocol uses 4 wires: *MOSI*, *SCK*, *D/C*, *CS* (*VCC* and *GND* aren't included).

After some quick googling I realized that you should add all resistors specified for an interface to enable it.
I decided to use SPI at first. I didn't use I2C and I didn't want to deal with it that time. All the more so *R3* was already soldered, and *R1* and *R8* were absent.

I took an Arduino Nano clone:

[MINI USB Nano V3.0 ATmega328P CH340G 5V 16M Micro-controller board for Arduino](https://rover.ebay.com/rover/1/711-53200-19255-0/1?icep_id=114&ipn=icep&toolid=20004&campid=5338194349&mpre=http%3A%2F%2Fwww.ebay.com%2Fitm%2FMINI-USB-Nano-V3-0-ATmega328P-CH340G-5V-16M-Micro-controller-board-for-Arduino-%2F381374550571)

![Arduino Nano Clone](/assets/03_funduino_nano.jpg)

Well, I could not find *MOSI* and *SCK*. I had *D0* и *D1* instead. It was time to look into the Chinese archive. Great, there was a file (with a name where I recognized only '.txt') which described the connnector's pinout!

The content is here without any change (looks mysterious, ain't it?):

```
	½ÓÏßËµÃ÷£º
 
   GND              µØ
   VCC              3.3V»òÕß5V
   D0(OLED_CLK)     10
   D(OLED_MOSI)    9
   RES(OLED_RESET)  13
   D/C(OLED_DC)      11
   CS£¨OLED_CS£©    12
```

Well, I connected it as described in that file:

![OLED to Arduino connection](/assets/04_oled_connected.jpg)

I didn't trust a library which was kindly provided by the Chinese seller. So that I used `SSD1306` library from a well-known Adafruit:

[https://github.com/adafruit/Adafruit_SSD1306](https://github.com/adafruit/Adafruit_SSD1306).

Library should be added into Arduino IDE, I am not going to describe it here.

I used an example: `ssd1306_128x64_spi`, which was corresponding to the display's resolution and interface. A firmware was uploaded, and... Hooray, it displayed! Unfortunately not something I was expecting for:

![Improper function with Adafruit Library](/assets/05_oled_failure.jpg)

Well, looks like there was some reason why the Chinese seller had put their library in the archive. I added the library, uploaded their example... Result was the same.

Weird, a part of screen worked OK but it had some noise at other part. But the normal area was pretty rectangular... Addresing issue? Broken memory? The ribbon cable was soldered not too accurate. Maybe some lines had miscontact? I watched it carefully — yes, they are not too accurate but they should have good contact...

I don't remember what exact keywords brought me to success. They were like "OLED failure partial screen visible» or some kind of. I found the following topic at **arduino.cc**:
[OLED 1.3" I2C IIC 128x64 Serial LCD — Faulty?](http://forum.arduino.cc/index.php?topic=259186.0)

That topic was about I2C display but an image from there looks like what I got:
![Improper function with Adafruit Library from arduino.cc](/assets/06_oled_failure_from_arduino_cc.jpg)

The topic contained some info that such displays could be supplied with different controllers. For example, it could be `SH1106` as it was for the topic-starter. The Adafruit library couldn't work with such displays and showed the similar artifacts.
But the library [u8glib](https://github.com/olikraus/u8glib) supported them.

I added that library to Arduino IDE and uploaded an example — "HelloWorld" or some kind of.

Every example contains different commented out options for connecting to different controllers using different screen resolutions, protocols, etc.
<br>
I needed the following line:

```
//U8GLIB_SH1106_128X64 u8g(13, 11, 10, 9);	// SW SPI Com: SCK = 13, MOSI = 11, CS = 10, A0 = 9
```
I uncommented it. Anyway, there was something wrong in it. What is a pin *A0*? I saw that label on other displays but which pin is corresponding on mine one? 

I found a clue that *A0* was *D/C* in u8glib wiki (that time on google code not github).
I connected all the pins that way they were corresponding to the code line above. Uploaded a firmware... Nothing happened. Not a single pixel was lit.

But arduino'cc topic-starter had managed to make it work! And the display was apparently good as it was able to show something. I should have further attempts.

Well, the SSD1306 library had shown something... I thought I needed to connect it the same way. Done. But where should I put a "reset"?
I saw some commented initialization lines with "reset" for some controllers which were unknown for me.
Looks like it is possible. I looked through the library source code. Yeah!

This is it:

```
U8GLIB_SH1106_128X64(uint8_t sck, uint8_t mosi, uint8_t cs, uint8_t a0, uint8_t reset = U8G_PIN_NONE) 
```

reset — is an optional pin. And I would specify it. The rest pins would be specified as in the example from Adafruit library:

```
U8GLIB_SH1106_128X64 u8g(10, 9, 12, 11, 13); // SW SPI Com: SCK = 10, MOSI = 9, CS = 12, D/C = 11, RESET = 13
```
Uploaded a firmware... Got it!

![Display shows properly](/assets/07_oled_ok.jpg)

Then I uploaded another example. I liked a "FPS" example very much, the display refreshed pretty fast.

## Summary

- Never give up, especially if someone has already managed to make the same stuff work.
- It is more reasonable to purchase a display with single protocol — only I2C or SPI.
- Don't trust much to information received from Chinese sellers.
- It is more reasonable to buy displays with pinouts matching to original displays' pinout from Adafruit and other major suppliers.
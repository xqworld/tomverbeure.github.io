---
layout: post
title: Monoprice Blackbird 4K Pro HDCP Converter
date:  2022-11-24 00:00:00 -1000
categories:
---

* TOC
{:toc}

# Introduction

I got my hands on a 
[Monoprice Blackbird 4K Pro HDCP 2.2 to 1.4 Converter](https://www.monoprice.com/product?p_id=15242).
According to the marketing copy it "is the definitive solution for playback of new 4K HDCP 2.2 encoded 
content on 4K displays with the old HDCP 1.4 standard."

Stuffed after a delicious Thanksgiving meal, I decided to take it apart after the guests had left. It's
a simple single-function device, so I didn't expect much, but maybe there's some things to be learned?

# Inside the Monoprice Blackbird 4K Pro

The outside of the Blackbird is forgettable, with the kind of black metal case that's used by most
protocol conversion boxes.

![Top view](/assets/hdcp_converter/top_view.jpg)

![Bottom view](/assets/hdcp_converter/bottom_view.jpg)

After removing the top cover, you're greeted with the following:

[![Inside view - annotated](/assets/hdcp_converter/inside_view annotated.jpg)](/assets/hdcp_converter/inside_view annotated.jpg)
*(Click to enlarge)*

As expected, the design is straightforward: the main conversion IC, an SPI flash memory, a microcontroller, 
and a bunch of jellybean components to create power rails.

Let's look at the main actors:

* Silicon Image Sil9679CNUC 

    Google has quite a bit of hits for this product, but they're all articles from way back when HDCP 2.2
    capable televisions were rare. Apparently, this chip was the very first one that allowed televsion 
    manufacturers to add HDCP 2.2 to their 4K TVs, though with the significant limitation that it can
    only do YUV 4:2:0.

    A few years ago, Silicon Image was acquired by Lattice Semiconductor. The Sil9679 is listed on Lattice 
    [TV HDMI / MHL Receivers product page](https://www.latticesemi.com/en/Products/ASSPs/TVHDMIMHLReceivers):

    [![Lattice Product Page](/assets/hdcp_converter/lattice_product_page.png)](/assets/hdcp_converter/lattice_product_page.png)
    *(Click to enlarge)*

    But there's no datasheet to be found.

    However, the Google search for "Sil9679 datasheet" turns up a 
    [datasheet for the Sil9678](https://datasheet.lcsc.com/lcsc/1912111437_Lattice-SiI9678CNUC_C369587.pdf) 
    on LCSC. It's a different product that does exactly the opposite: convert from HDCP 1.4 to HDCP 2.2, but
    it has the same packages and after close examination of the PCB, the pinout seems identical at well!

    Excellent!

    Chances are high that the Sil9678 and Sil9679 use the same silicon with different functions 
    fused off...

* STMicroelectronics STM8S005K6T6C Microcontroller

    This part has no secrets: the [datasheet](/assets/hdcp_converter/stm8s005c6.pdf)
    is readily available.

    This member of the [STM8 series](https://en.wikipedia.org/wiki/STM8)has an 8-bit CPU, 32KB of flash memory, 
    2KB of RAM, and the usual assortment of timers, UART, I2C, and SPI interfaces. It's just what you need for 
    some simple configuration and control of another chip.

* Console Connector J1

    You can visually trace the PCB traces from connector J1 to the STM8 controller: 2 pins are connected
    to its UART interface. We're definitely going to have a closer look at that!

* Debug Connector J2

    The second pin of connector J2 is connected to the SWIM pin of the STM8. SWIM is short for
    [Single Wire Interface Module](http://kuku.eu.org/?projects/stm8spi/stm8spi#mcb_toc_head1). 
    It's a single-pin equivalent of the more common JTAG interface.


# Tests done

* Basic plan: 
    * Linux machine through converter: no HDCP seen. Works
    * Shield TV to HDCP 1.4 TV through converter: HDCP 2 on source, HDCP 1 on TV. Works.

* Premium plan:
    * Linux machine: no HDCP seen. Only HD content. Works.
    * Shield TV directly to HDPC 2 monitor: 4K and HDR content. Works.
    * Shield TV to HDPC 2 monitor via down converter: Black screen. Converter doesn't support 444?
    * Shield TV to HDPC 2 monitor via down converter in 1080p and HDR. Works! Is this Type 1 or Type 0 content???
    * Shield TV directly to old TV: 1080p HD only. No HDR listed.

# Components

* Sil9679CNUC (or was it 9879CNDC?)

    * [Product page](https://www.latticesemi.com/Products/ASSPs/TVHDMIMHLReceivers)
    * 76 pin QFN
    * [Datasheet for Sil9678](https://datasheet.lcsc.com/lcsc/1912111437_Lattice-SiI9678CNUC_C369587.pdf)
	* Converts from HDCP 1.4 to HDCP 2.2 instead of the opposite?

* STM8S005K6T6C

    * [Datasheet](/assets/hdcp_converter/stm8s005c6.pdf)

* J1: connects to STM UART pins 30 and 31
* J2: connects to STM SWIM (single wire interface module) pin 26
* SPI chip connected to Sil

* Difference between Type 0 and Type 1 content


# Messages

Bootup message:

```
        Copyright Silicon Image Inc, 2013
        Build Time: Aug 31 2015-14:50:30

        Host Software Version: 01.02.02
        Adapter Driver Version: 01.00.11
        Require FW version: 02.02.xx

Waiting Boot Done  ....
Booting is done successfully.

Chip ID: 0x9679
Firmware Version: 01.02.06
***Error: Chip FW Version not match.
```

Plug in sink:

```
Downstream Connect Status Changed:
   Link Type is: HDMI/DVI

Tx HDCP  Version:
   HDCP version is: 1.x

Tx HDCP Status Changed:
   Tx HDCP Authentication Off.

Rx HDCP  Version:
   HDCP version not support

Rx HDCP Status Changed:
   Rx HDCP Authentication Off.


Edid Status: Avaliable
Edid Block 0 Data:
00 ff ff ff ff ff ff 00 24 62 00 32 01 01 01 01 
10 1c 01 03 80 47 28 78 3e 64 35 a5 54 4f 9e 27 
12 50 54 bf ef 80 d1 c0 b3 00 a9 c0 95 00 90 40 
81 80 81 40 81 c0 40 d0 00 a0 f0 70 3e 80 30 20 
35 00 54 4f 21 00 00 1a b4 66 00 a0 f0 70 1f 80 
30 20 35 00 80 88 42 00 00 1a 00 00 00 fc 00 48 
44 4d 49 32 0a 20 20 20 20 20 20 20 00 00 00 fd 
00 18 90 0f de 3c 00 0a 20 20 20 20 20 20 01 30 
Edid Block 1 Data:
02 03 4f f2 5e 04 05 10 13 14 1f 20 21 22 27 48 
49 4a 4b 4c 5d 5e 5f 60 61 62 63 64 65 66 67 68 
69 6a 6b e2 00 c0 e3 05 c0 00 23 09 7f 07 83 01 
00 00 e5 0f 00 00 0c 00 67 03 0c 00 10 00 38 78 
67 d8 5d c4 01 78 88 01 e6 06 05 01 69 69 4f 02 
3a 80 18 71 38 2d 40 58 2c 25 00 55 50 21 00 00 
1e 69 5e 00 a0 a0 a0 29 50 30 20 35 00 00 b0 31 
00 00 1e 00 00 00 00 00 00 00 00 00 00 00 00 df 

```

Plug in active HDCP 1.4 source:

```
Upstream Connect Status Changed:
   Link Type is: HDMI/DVI


Sync Detect: ON

Clock Detect: ON


AV Mute Status Changed:
   AV Mute ON

Tx HDCP  Version:
   HDCP version is: 1.x

Tx HDCP Status Changed:
   Tx HDCP Authenticating...

Rx HDCP  Version:
   HDCP version is: 1.x

Rx HDCP Status Changed:
   Rx HDCP Authentication Done.

Rx HDCP  Version:
   HDCP version is: 1.x

Rx HDCP Status Changed:
   Rx HDCP Authentication Done.


Tx HDCP  Version:
   HDCP version is: 1.x

Tx HDCP Status Changed:
   Tx HDCP Authentication Off.

Rx HDCP  Version:
   HDCP version is: 1.x

Rx HDCP Status Changed:
   Rx HDCP Authentication Done.

```

HDCP 1.4 source goes to sleep:

```
Sync Detect: OFF

AV Mute Status Changed:
   AV Mute OFF

Tx HDCP  Version:
   HDCP version is: 1.x

Tx HDCP Status Changed:
   Tx HDCP Authentication Off.

Rx HDCP  Version:
   HDCP version not support

Rx HDCP Status Changed:
   Rx HDCP Authentication Off.


Clock Detect: OFF
```



* [Reposted: A cryptanalysis of HDCP v2.1](https://blog.cryptographyengineering.com/2012/08/27/reposted-cryptanalysis-of-hdcp-v2/)
* [hdcp-genkey](https://github.com/rjw57/hdcp-genkey)
* [HDCP Specifications](https://www.digital-cp.com/hdcp-specifications)

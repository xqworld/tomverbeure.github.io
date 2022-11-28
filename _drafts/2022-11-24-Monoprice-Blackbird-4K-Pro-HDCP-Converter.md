---
layout: post
title: Monoprice HDCP 2.2 to 1.4 Down Converter Protocol Analysis
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

# Some Words about HDCP

HDCP ([High-Bandwidth Digital Content Protection](https://en.wikipedia.org/wiki/High-bandwidth_Digital_Content_Protection))
version 1 dates from somewhere at the beginning of the century and turned out to be terrible broken.
By 2010, it was completely defeated with the release of the 
[master key](https://github.com/rjw57/hdcp-genkey/blob/master/master-key.txt)[^1] that can be used to 
generate transmit and receive keys at will.

To keep Hollywood happy, a succesor was invented: HDCP 2.0, an entirely new protocol with no similarities 
to revision 1.  This time, the creators decided use to standard cryptographic algorithms such as RSA public key
authentication, and 128-bit AES encryption. HDCP 2.1 added the concept of content stream types to block the 
retransmition of high quality video streams (e.g. 4K or HDR) to a lower level of protection.

Despite using known-good core algorithms, there was still 
[a major issue](https://blog.cryptographyengineering.com/2012/08/27/reposted-cryptanalysis-of-hdcp-v2/)
in the way certain data was exchanged between transmitter and receiver, necessitating a 2.2 revision
that wasn't backward compatible with versions 2.0 and 2.1. There's an HDCP version 2.3 now, but
it's not reallly clear if there are any real-world consequences...

Today, all new TVs support both HDCP 2 and HDCP 1 so that legacy HDCP 1-only sources such as DVD players 
are still supported. But that wasn't the case initially: the 15 year old 1080p TV in our house dates from well 
before the introduction of HDCP 2. Luckily, most modern video transmitters (in our case the
excellent Nvidia Shield TV media play with a brilliant command DMA engine) still support HDCP 1 fine.

There are some issues however: remember how HDCP 2.1 added content stream types? There only 2 of
them: type 0 and type 1, where type 1 is considered high value. It's up to a content provider to decide 
whether a video stream gets tagged as such. In the case of Netflix, UHD (4K) and HDR content is classified as 
type 1 while traditional HD content (720p and 1080p non-HDR) is type 0.

If you have a 4K monitor that only supports HDCP 1, you're out of luck: Netflix will only serve
1080p streams. The issue can't be solved with repeaters such as home theater video receivers, because 
the HDCP specification prohibits the downconversion of HDCP2 to HDCP1 for type 1 content.

And yet, there are devices like that Monoprice Blackbird that claimed to be down this conversion just
fine? Maybe it only supports type 0 content, making it technically HDCP 2 compliant even if it doesn't
supports only a fraction of the available content? 

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
    capable TV silicon was rare. Apparently, this chip was the very first one that allowed televsion 
    manufacturers to add support for HDCP 2.2 to their 4K TVs, though with the significant limitation that it 
    can only do YUV 4:2:0.

    A few years ago, Silicon Image was acquired by Lattice Semiconductor. The Sil9679 is listed on Lattice 
    [TV HDMI / MHL Receivers product page](https://www.latticesemi.com/en/Products/ASSPs/TVHDMIMHLReceivers):

    [![Lattice Product Page](/assets/hdcp_converter/lattice_product_page.png)](/assets/hdcp_converter/lattice_product_page.png)
    *(Click to enlarge)*

    But there's no datasheet to be found.

    BTW, notice how the table above lists HDMI 2.0 support. Most people associate HDMI 2.0 with full quality 4K 60Hz
    support, which requires a cable bandwidth above the limit of HDMI 1.4. 4K 60Hz in  YUV 4:2:0 mode, however,
    fits nicely within the capabilities of HDMI 1.4. It's a mystery why the table claims HDMI 2.0 capabilities...[^2]

    A Google search for "Sil9679 datasheet" turns up a 
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

# HDCP Details

* [HDCP on HDMI Specification](https://www.digital-cp.com/sites/default/files/HDCP%20on%20HDMI%20Specification%20Rev2_3.pdf)
  section 3.2 HDCP Cipher: content type is not part of cipher operation.
* [HDCP on DisplayPort Specification](https://www.digital-cp.com/sites/default/files/specifications/HDCP%20on%20DisplayPort%20Specification%20Rev2_2.pdf)
   section 3.2 HDCP Cipher: 8-bit content type is XORed with riv to create initialization vector.

# References

* [HDCP Specifications](https://www.digital-cp.com/hdcp-specifications)
* [A Cryptanalysis of the High-bandwidth Digital Content Protection System](https://cypherpunks.ca/~iang/pubs/hdcp-drm01.pdf)
* [A cryptanalysis of HDCP v2.1](https://blog.cryptographyengineering.com/2012/08/27/reposted-cryptanalysis-of-hdcp-v2/)
* [hdcp-genkey](https://github.com/rjw57/hdcp-genkey)

# Footnotes

[^1]: It wasn't necessary for someone in-the-know to leak the master key, the protocol was so flawed
      that the master key can be derived using the technique described in 
      "A Cryptanalysis of the High-bandwidth Digital Content Protection System" by Crosby, et al.

[^2]: You can the bandwidth requirements with my [video timings calculator](https://tomverbeure.github.io/video_timings_calculator).
      Just chose 4K/60 in the predefined mode selection box, and toggle between the RGB 4:4:4 or YUV 4:2:0 color formats.
      You'll see how RGB 4:4:4 requires HDMI 2.0 while YUV 4:2:0 works with HDMI 1.4.

      ![Video Timings Calculator](/assets/hdcp_converter/video_timings_calculator.png)

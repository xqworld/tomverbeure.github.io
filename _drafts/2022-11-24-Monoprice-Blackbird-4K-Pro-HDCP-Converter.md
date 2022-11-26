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

![Inside view - annotated](/assets/hdcp_converter/inside_view annotated.jpg)

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

---
layout: post
title: Monoprice Blackbird 4K Pro HDCP Converter
date:  2022-11-24 00:00:00 -1000
categories:
---

* TOC
{:toc}


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

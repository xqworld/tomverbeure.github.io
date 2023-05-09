---
layout: post
title: DSLogic U3Pro16 Teardown
date:   2023-03-20 00:00:00 -1000
categories:
---

* TOC
{:toc}

# Introduction

The effort of converting a captured printer trace ranges from entirely painless to
very painful. It depends on which printer formats are supported by your instrument, and
the desired quality.

Components:

* FPGA: Spartan-6 XC6SLX16-FTG256DIV1945
* DRAM: DDR3-1600: Micron MT41K128M16JT-125 (D9PTK), 2Gb, 16-bit
* DDR termination voltage controller: [SGM2054](http://www.sg-micro.com/uploads/soft/20220506/1651829741.pdf)
* FPGA power regulator: [LM26480](https://www.ti.com/lit/ds/symlink/lm26480.pdf)
* 24MHz oscillator
* USB interface: CYSB3014-BZX
    * [$14 on LCSC](https://www.lcsc.com/product-detail/Microcontroller-Units-MCUs-MPUs-SOCs_Cypress-Semicon-CYUSB3014-BZXC_C462526.html)
* 19.2MHz oscillator
* USB ESD protection: SP3010-04UTG
    * marked QH4
    
* USB-C mux: TI [HD3SS3220](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
* Clock generator: [ADF4360-7](https://www.analog.com/media/en/technical-documentation/data-sheets/ADF4360-7.pdf)
* Flash: Macronix [MX25R2035F](https://www.macronix.com/Lists/Datasheet/Attachments/8696/MX25R2035F,%20Wide%20Range,%202Mb,%20v1.6.pdf)
* Measurement ESD protection: [SRV05-4D-TP](https://www.mccsemi.com/pdf/Products/SRV05-4D(SOT23-6L).pdf)

    Probably! Might be slightly different...

* 100k resistor to ground + ESD -> 33Ohm series resistor (into FPGA?)

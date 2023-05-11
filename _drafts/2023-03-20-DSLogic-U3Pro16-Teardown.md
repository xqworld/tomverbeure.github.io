---
layout: post
title: DSLogic U3Pro16 Teardown
date:   2023-03-20 00:00:00 -1000
categories:
---

* TOC
{:toc}

# Introduction

The year was 2020, and the offices all over the world shut down. A house remodel had just
started, so work moved from a comfortably airconditioned room to a very messy garage.

[![A very messy garage](/assets/dslogic/garage.jpg)](/assets/dslogic/garage.jpg)

Since I'm in the business of developing, and debugging, hardware, a few bit of
work equipment went along for the ride, including a Saleae Logic Pro 16. There's
no way around it: they're at the top of USB logic analyzers. Plenty of competitors
have matched or surpassed their digital features, but none have the ability to record 
the 16 channels in analog format as well.

I obviously used the Saleae for work stuff, but I may once in a while also have used
it for some hobby-related activities.

![Probing the pins of an HP 3478A multimeter](/assets/hp3478a/sram_with_probes.jpg)

Last September, corporate offices opened up again, the Saleae went back to its
original habitat, and I found myself without a good 16-channel USB logic analyzer.

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

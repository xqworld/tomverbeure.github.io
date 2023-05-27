---
layout: post
title: DSLogic U3Pro16 Review and Teardown
date:   2023-03-20 00:00:00 -1000
categories:
---

* TOC
{:toc}

# Introduction

The year was 2020, and the offices all over the world shut down. A house remodel had just
started, so office moved from a comfortably airconditioned corporate building to a very messy 
garage.

[![A very messy garage](/assets/dslogic/garage.jpg)](/assets/dslogic/garage.jpg)

Since I'm in the business of developing, and debugging, hardware, a few items of
work equipment came along for the ride, including a Saleae Logic Pro 16. There's
no way around it: they're at the top of USB logic analyzers. Plenty of competitors
have matched or surpassed their digital features, but none have the ability to record 
the 16 channels in analog format as well.

I obviously had the Saleae for work stuff, but I may once in a while also have used
it for some hobby-related activities.

![Probing the pins of an HP 3478A multimeter](/assets/hp3478a/sram_with_probes.jpg)

Last September, corporate offices opened up again, the Saleae went back to its
original habitat, and I found myself without a good 16-channel USB logic analyzer.

After looking around for a while, I decided to give the 
[DSLogic U3Pro16](https://www.dreamsourcelab.com/product/dslogic-series/)
from [DreamSourceLab](https://www.dreamsourcelab.com) a chance. I bought it 
[on Amazon](https://www.amazon.com/DreamSourceLab-USB-Based-Analyzer-Sampling-Interface/dp/B08C2LCBGL/ref=sr_1_3) 
for $299.

[![DSLogic with all probe wires](/assets/dslogic/dslogic_with_all_wires.jpg)](/assets/dslogic/dslogic_with_all_wires.jpg)
*(Click to enlarge)*

In this blog post, I'll look at some of the features, my experience with the software,
and I'll also open it up to discover what's inside.

# The DSLogic U3Pro16

DreamSourceLab currently sells 3 logic analyzers: 

* the $149 DSLogic Plus
* the $299 DSLogic U3Pro16
* the $399 DSLogic U3Pro32

The only functional difference between the U3Pro16 and U3Pro32 is the number of channels; 
they're otherwise identical. It's tempting to go for the 32 channel version  but I've rarely 
had the need to record more than 16 channels and if I really need it, I can always fall back 
to my HP 1670G logic analyzer, a pristine $200 flea market treasure with a whopping 136 channels.

[![HP 1670G](/assets/dslogic/hp1670g.jpg)](/assets/dslogic/hp1670g.jpg)

The DSLogic Plus also has 16 channels, but its acquisition memory is only 256Mbits vs 2Gbits
for the U3Pro16, and, more important, it has to make do with USB 2.0 instead of a USB 3.0
interface. That's a crucial difference when streaming acquistion data straight to the PC, to
avoid the limitations of the acquistion memory. There's also a difference in sample
rate, 400MHz for the Plus, 1GHz for the U3Pro16, but that's not a major difference in
practice.

# In the Box

The DSLogic comes with a nice, elongated hard case. 

[![DSLogic case](/assets/dslogic/dslogic_case.jpg)](/assets/dslogic/dslogic_case.jpg)

Inside, you'll find the following:

* the device itself, a slick aluminum case
* a USB-C to USB-A adaptor cable
* 5 4-way probe cables and 1 3-way clock and trigger cable
* 16 test clips

Yes, you read it right, 5 4-way probe cables, not 4. I don't know if DreamSourceLab adds one
extra in case you lose one, or if they mistakenly included one to much, but it's good to
have a spare one.

[![DSLogic contents](/assets/dslogic/dslogic_contents.jpg)](/assets/dslogic/dslogic_contents.jpg)

The cables are quite stiff, we'll get to that later, and are definitely not as pliable as the
ones that comes with a Saleae. The case has been designed such that the probe cables can
be stored without the need to bend them. I like it.

The quality of the test clips is bad, but no different than those of a $1500(!) Saleae.








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

# References

*Reviews*

* [Logic Analyzer Shopping](https://www.bigmessowires.com/2021/11/21/logic-analyzer-shopping/)

    Comparison between Saleae Logic Pro 16, Innomaker LA2016, Innomaker LA5016, DSLogic Plus, and DSLogic U3Pro16



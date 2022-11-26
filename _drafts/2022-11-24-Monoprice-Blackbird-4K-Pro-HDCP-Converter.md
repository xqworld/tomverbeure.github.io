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

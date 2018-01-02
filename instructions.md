# 6LoWPAN notes

## Required

* Border router.
* Mbed OS 5 Development board - https://os.mbed.com/platforms/?mbed-os=25
* 6LoWPAN shield (802.15.4 shield)
    * NXP MCR20 - https://www.nxp.com/support/developer-resources/hardware-development-tools/freedom-development-boards/wireless-connectivy/freedom-development-board-for-mcr20a-wireless-transceiver:FRDM-CR20A
    * STM Spirit1 - http://www.st.com/en/wireless-connectivity/spirit1.html
    * ATMEL AT233 / ATMEL AT86RF212B - https://l-tek.si/web-shop/l-tek-6lowpan-arduino-shield-2-4ghz/ (also in Sub-GHz).
    * SiLabs EFR32 (Flex Gecko) - https://www.silabs.com/documents/public/reference-manuals/brd4252a-rm.pdf

6LoWPAN can be deployed on a variety of frequencies, border router and shield should match frequencies. E.g. 2.4 GHz or sub-GHz. You can see the difference based on the antenna. Some additional background on choosing sub-GHz vs 2.4 GHz would be good, e.g. deployment worldwide (2.4 GHz), longer range (sub-GHz). Perhaps add some link budget numbers.

**Link budget**

[Atmel AT86RF233](http://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-8351-MCU_Wireless-AT86RF233_Datasheet.pdf) (2.4 GHz) has -101 dBm RX sensitivity. [Atmel AT86RF212B](http://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-42002-MCU_Wireless-AT86RF212B_Datasheet.pdf) (900 MHz) has -110 dBm RX sensitivity. Note that these are max. numbers, and dependent on the speed of transmission / modulation scheme. Sub-GHz will go further, plus additionally longer waves will travel further regardless ([path loss calculator](http://www.wirelesscommunication.nl/reference/chaptr03/pel/loss.htm)).

**Border routers**

A border router is essentially a normal node, but with connectivity to the outside world. It also needs power. The border router needs to run some routing software. This is located at [nanostack-border-router](https://github.com/ARMmbed/nanostack-border-router). Like a normal node it's a combination of microcontroller + 6LoWPAN radio. However, it's better to just buy an off the shelf border router, as it's certified and comes in a box. L-TEK has the BeeConn border router, which is used in this article, which essentially is a FRDM-K64F + Atmel chip.

A list of targets that work are [here](https://github.com/ARMmbed/mbed-os-example-mesh-minimal/blob/master/Hardware.md).

**IPv6**

The border router needs access to an IPv6 network to work (because 6LoWPAN is IPv6). If you don't have access to IPv6 network you can use a tunneling service, such as [Hurricane Electric](https://tunnelbroker.net) but it's out of scope for the article (I think). **<-- I think this might actually be valuable information... But I don't have good resources.**

**Software requirements**

Mbed OS applications can be built in the Online Compiler and through Mbed CLI. Instructions on both of them are at https://os.mbed.com/docs/v5.6/tools/index.html.

## Building border router software

This is done with the BeeConn border router from L-TEK. It comes configured as a thread border router, but I'd rather have a 6LoWPAN border router.

First connect the BeeConn border router to your computer using a USB cable.

**Mbed CLI**

```
$ mbed import https://github.com/ARMmbed/nanostack-border-router
$ cd nanostack-border-router
$ cp configs/6lowpan_Atmel_RF.json mbed_app.json
$ mbed compile -m K64F -t GCC_ARM
```

Go into the `BUILD/K64F/GCC_ARM` folder, and locate `nanostack-border-router.bin`. Drag it to the BeeConn device - which mounts as USB mass-storage device when connecting to your computer.

**Online Compiler**

1. Go to [nanostack-border-router](https://os.mbed.com/teams/mbed-os-examples/code/nanostack-border-router/).
1. Click *Import into Compiler*.
1. In the compiler click on the target (top right corner), click *Add new target*.
1. Search for FRDM-K64F, and click *Add to compiler*.
1. Go back to the compiler and select FRDM-K64F as target.
1. Copy `configs/6lowpan_Atmel_RF.json` to `mbed_app.json`.
1. Click *Compile*.

A file downloads. Drag it to the BeeConn device - which mounts as USB mass-storage device when connecting to your computer.

**Inspecting logs on the device**

Connect a serial monitor to the device on baud rate 115,200. It should show something similar to:

```
[INFO][brro]: PANID: 691
[INFO][brro]: NET_IPV6_BOOTSTRAP_AUTONOMOUS
[WARN][brro]: Security NOT enabled
[INFO][app ]: Starting NanoStack Border Router...
[INFO][app ]: Build date: Dec 19 2017 12:06:27
[INFO][app ]: Using ETH backhaul driver...
[INFO][Eth ]: Ethernet cable connected.
[INFO][addr]: Tentative Address added to IF 2: fe80::5843:4aff:fe2e:8a21
[INFO][addr]: DAD passed on IF 2: fe80::5843:4aff:fe2e:8a21
[INFO][addr]: Tentative Address added to IF 2: fd60:5ff5:15e3:0:5843:4aff:fe2e:8a21
[INFO][addr]: DAD passed on IF 2: fd60:5ff5:15e3:0:5843:4aff:fe2e:8a21
[INFO][brro]: Backhaul bootstrap ready, IPv6 = fd60:5ff5:15e3:0:5843:4aff:fe2e:8a21
[INFO][brro]: Backhaul interface addresses:
[INFO][brro]:    [0] fe80::5843:4aff:fe2e:8a21
[INFO][brro]:    [1] fd60:5ff5:15e3:0:5843:4aff:fe2e:8a21
[INFO][brro]: RF channel: 12
[INFO][br  ]: BR nwk base ready for start
[INFO][addr]: Address added to IF 1: fe80::ff:fe00:face
[INFO][br  ]: Refresh Contexts
[INFO][br  ]: Refresh Prefixs
[INFO][addr]: Address added to IF 1: fd60:5ff5:15e3::ff:fe00:face
[INFO][addr]: Address added to IF 1: fe80::fec2:3d00:3:7aef
[INFO][brro]: RF bootstrap ready, IPv6 = fd60:5ff5:15e3::ff:fe00:face
[INFO][brro]: RF interface addresses:
[INFO][brro]:    [0] fe80::ff:fe00:face
[INFO][brro]:    [1] fe80::fec2:3d00:3:7aef
[INFO][brro]:    [2] fd60:5ff5:15e3::ff:fe00:face
[INFO][brro]: 6LoWPAN Border Router Bootstrap Complete.
```

Now the border router is online.

**Channel selection**

Both device and gateway need to use the same channel. It's configured in `mbed_app.json`, see the `rf-channel` setting. Default setting is 12.

## Connecting a device

Multiple devices broadcast over the same network.

* [mbed-os-example-mesh-minimal](https://github.com/ARMmbed/mbed-os-example-mesh-minimal).
* [Video](img/6lowpan-two-devices-mesh.MOV) - using two different dev boards and two different radio chips.

How to:

1. Clone the example.
2. Pick connectivity method in mbed_app.json.
3. Select LED and BUTTON to control the LEDs over broadcast.
4. Compile.

**Drivers**

* NXP MCR20 - https://github.com/ARMmbed/mcr20a-rf-driver/
* STM Spirit1 - https://github.com/ARMmbed/stm-spirit1-rf-driver/
* ATMEL AT233 - https://github.com/ARMmbed/atmel-rf-driver/
* SiLabs EFR32 - https://github.com/ARMmbed/mbed-os/tree/master/targets/TARGET_Silicon_Labs/TARGET_SL_RAIL/efr32-rf-driver (already included in Mbed OS).

## Connecting to the internet

HTTP and HTTPS calls over 6LoWPAN.

* Import [http-example](https://os.mbed.com/teams/sandbox/code/http-example/).
* Open `mbed_app.json`, change `network-interface` into `MESH_LOWPAN_ND`.
* In `mbed_app.json`, also set your 6LoWPAN radio (under `mesh_radio_type`).
* Open `source/select-demo.h` and select `DEMO_HTTP_IPV6`.
* Compile and run the application. It will show something like:

    ```
    [EasyConnect] IPv6 mode
    [EasyConnect] Using Mesh (Atmel)
    [EasyConnect] Connecting to Mesh...
    [EasyConnect] Connected to Network successfully
    [EasyConnect] MAC address fc:c2:3d:00:00:04:e4:a3
    [EasyConnect] IP address 2001:470:1d4c:1:fec2:3d00:4:e4a3

    ----- HTTP GET response -----
    Status: 200 - OK
    Headers:
            Server: nginx
            Date: Tue, 02 Jan 2018 15:18:59 GMT
            Content-Type: text/plain; charset=UTF-8
            Content-Length: 33
            Connection: close
            X-SECURITY: This site DOES NOT distribute malware. Get the facts. https://goo.gl/1FhVpg
            X-RTFM: Learn about this site at http://bit.ly/icanhazip-faq and do not abuse the service.
            Access-Control-Allow-Origin: *
            Access-Control-Allow-Methods: GET

    Body (33 bytes):

    2001:470:1d4c:1:fec2:3d00:4:e4a3
    ```

* The body contains the IPv6 address of your device.

**Note:** Nanostack (the underlying IP stack for Mbed's mesh tech) does not support IPv4. Only IPv6.

mbed-http also supports HTTPS. See demo-https.cpp for more information. You need to add the CA to the list of trusted CAs.

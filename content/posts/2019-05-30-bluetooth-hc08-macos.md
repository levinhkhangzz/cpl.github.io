---
title: "HC-08 Bluetooth (LE) with Arduino and macOS"
description: "My experience with a HC-08 Bluetooth module and macOS BLE"
keywords: [bluetooth, macOS, arduino, hardware, BLE HC-08, apple]
date: 2019-05-30
---

I received one of these `HC-08` Bluetooth modules for Arduino and other micro controllers. I thought it will be a nice, fun and simple experience. After all everything has or used to have Bluetooth, we are in the age of “[IoT](https://twitter.com/internetofshit)”.

<div class="video-container">
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/0KXoBcQER_0?start=93" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

## HC-08

---

While it’s not the best Bluetooth module out there, it’s also not the worst. Connecting it to a micro controller is straightforward. Connect `GND` to `GND`, `VCC` to a `5V` power source, the `HC-08` `TX` to the controller `RX`. For the `RX` of the `HC-08`, you must watch out, as it requires `3.3V`. If your controller uses `5V` logic, then you must adapt your voltage to 3.3 (you can use the schematic below).

![Connecting HC-08 to Arduino Uno](/assets/images/bluetooth-101_bb.png)

(the 1kOhm resistor is between the RX-TX line and the 2kOhm resistor connects back to the ground)

I used the following code to send messages and *commands* to turn an LED on and off.

<script src="https://gist.github.com/cpl/610b6ca3ec9c20cdceb2eecbde51b076.js"></script>

By using a BLE Terminal App I was able to detect my device, connect to it and communicate both ways. I was unable to do the same from **macOS**, but more on this later.

## AT Commands

---

You can send special commands over serial to your bluetooth module. Some devices may require special pins to be powered in a certain way. Check the datagram of your device for more information. In **AT** mode you can send string with the prefix `AT+`. To test wether your devices supports and is in **AT** mode, sending the string `AT` will return an `OK` message.

For a list of available AT commands check your modules manual. Some devices have a *mini-AT* mode. Commands can be used to query data (`AT+NAME=?`) will return the name of the device. The same command can be used to set the name (`AT+NAME=FooBar`).

## macOS & Bluetooth

---

The idea behind my project was controlling different devices from my terminal	(because I use it more than anything else and I always have a terminal window open somewhere) using BLE. 

When scanning for Bluetooth devices on **macOS** my `HC-08` wouldn’t show up, but my iPhone and Raspberry Pi were able to pick it up. I checked the hardware on my Mac and the Bluetooth is `LMP Version: 4.0 (0x6)`. Which means it has BLE support.

I thought to myself that maybe normal users have no need for ALL random micro controllers with Bluetooth to show up in their discovery menu, so maybe the underlaying software was hiding something from me (as Apple software normally does >.>). So i tried this tool [blueutil](https://github.com/toy/blueutil). But scanning for my device or even trying to directly connect using its address didn’t work.

This is when my journey to the deep corners of Apple’s IOKit drivers and Bluetooth. Their developer [documentation](https://developer.apple.com/bluetooth/) mentions some different frameworks. Most apps and devices use the common **IOBluetooth** Framework. For BLE devices you must use [**CoreBluetooth**](https://developer.apple.com/documentation/corebluetooth)

### XPC Workaround

While searching for people who found a way to access the Bluetooth API without using **Swift** or **Objective-C**, I found this [repo](https://github.com/noble/noble). It’s a NodeJS library that uses XPC connections to communicate with the Bluetooth service/daemon. This is a nasty workaround because Apple changes the inner-workings of their software with each major update and without notice. This will result in breaking changes and having to patch your library with every update, resulting in a big library full of duct-tape.

### Binding the API

The other *workaround* which is actually cleaner is [noble-mac](https://github.com/Timeular/noble-mac) which binds some **Swift** and **Objective-C** functions to JavaScript. The **Swift** and **Objective-C** use the Framework API and the JavaScript only links the frameworks + binds the functions and allows for easy use. I wanted to do the same using **Go**, but frankly that would take quite a while to setup and even before that I would have to read about **Swift**, **Objective-C** and **IOKit**. Instead I have my own workaround

### Middleman

I will be using my Raspberry Pi to controller any Bluetooth device. Maybe even setup a web server or basic API to not require me to be connected over SSH or something else.

## Apple Resources

---

In my endless search for a solution I did stumble upon some interesting [tools](https://developer.apple.com/download/more/?=additional%20Tools) developed by Apple for Apple and other interested developers.

![Apple Additional Tools](/assets/images/apple-tools.png)

One of these tools is a **Bluetooth Explorer** which is able to scan for BLE devices, display them and even connect. If I had to guess all these apps were developed using their APIs and would have loved to have a command line interface that does the very same thing. I hate GUIs.

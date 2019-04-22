---
layout: post
title: iot door sensor
date: 2019-1-28 15:00:20 +0300
description: An open source, motorized table for cnc applications
img: iot_door/smart-space.jpg
tags: [iot, arduino, electronics]
author: Aaron Peterson
---
# Introduction
This was a quick project I threw together to monitor the makerspace's front door. We have a membership tier that doesn't provide 24x7 access, but allows them to come in whenever a full member is present. This sensor pinged a channel on our Slack workspace to notify members when the space's front door was unlocked. It also helped us officers realize when the door was left open!

This uses a adafruit huzzah esp8266 development board and a magnetic reed switch on the door. When the door opens, the reed switch closes and tells the esp8266 to connect to the wifi and send a message to Slack. The code has around a 30 second debounce in order to account for people opening and shutting the door quickly (moving stuff in and out)

I designed and lasercut a project box enclosure for it and assembled it above the door

![smart-space]({{site.baseurl}}/assets/img/iot_door/smart-space.jpg)



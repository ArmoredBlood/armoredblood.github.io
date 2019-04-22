---
layout: post
title: iot stock price ticker
date: 2018-12-25 15:00:20 +0300
description: A stock price ticker for my dad
img: stock_ticker/ticker-front.jpg
tags: [iot, particle, electronics]
author: Aaron Peterson
---
# Introduction
For the holidays, I made my dad an IOT Stock Price Ticker that he could add his own portfolio and watch its performance. It's based on a Particle Photon which queries a securities API, and displays the info on an LCD display. It will cycle through up to 20 stocks. He can open up the ip address at the bottom of the display on his phone's web browser and maintain a comma separated list of securities. The entire project is hot glued in place inside a laser cut, living hinged, wooden enclosure.

![stock-ticker-front]({{site.baseurl}}/assets/img/stock_ticker/ticker-front.jpg)

# Electronics
The whole project is soldered onto a custom prototype circuit board. This breaks out the power input to a separate micro-usb port to move any stress from cabling onto an easy to replace breakout board. Female headers also provide me the ability to swap the Photon out if it ever were to break or stop functioning. The LCD display is soldered to the female headers that correspond to the Photon's SPI pins.

![stock-ticker-inside]({{site.baseurl}}/assets/img/stock_ticker/ticker-inside.jpg)

# Code
Particle provides the Particle Cloud IDE which allows me to flash updates to the code remotely, so I don't need to make the trip to my dad's just to fix a bug. This code serves a small web page from the Photon that allows the user to maintain a comma separated string of securities that the Photon will loop through and query the prices for. It will parse the output of the securities' API and display it up on the screen, in differente colors based on the performance of the security.

The code can be found in my [gitlab repo](https://gitlab.com/armoredblood/photon-stock-ticker) if you are interested.

![stock-ticker-web]({{site.baseurl}}/assets/img/stock_ticker/ticker-web-page.jpg)



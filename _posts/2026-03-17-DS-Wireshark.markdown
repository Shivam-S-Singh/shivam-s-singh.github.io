---
layout: posts
title:  "3DS File Transfer Failure"
date:   2026-03-17 21:52:57 -0400
tags: [Packet Analysis]
categories: work
highlight_home: true
tagline: "From capturing pokemon to capturing packets"
description: This article covers a packet analysis exercise on a File Transfer application.
author_profile: true
author: Shivam Singh
show_date: true
read_time: true
classes: wide
header:
  overlay_image: /assets/images/3DS_Analysis/nintendo_cover.jfif
  teaser: /assets/images/3DS_Analysis/nintendo_cover.jfif
  caption: "Photo Credit: [Nintendo]"
---

# Packet analysis of file transfer application used in Nintendo 3DS

> This article builds on a LinkedIn post I previously published, revisiting the exercise in a more detailed and structured format. You can view the original post [_here_](https://www.linkedin.com/feed/update/urn:li:activity:7439851307710017536/).

[![My 3DS](/assets/images/3DS_Analysis/3DS_Wireshark.jpg){: width="83%" style="display: block; margin: 0 auto;"}](/assets/images/3DS_Analysis/3DS_Wireshark.jpg)

## Overview
I wanted to backup the game data from my MicroSD card on my 3DS to my PC. The 3DS has a file transfer application called __microSD Management__ which lets you access and transfer your files from a _Windows 7_ or _8.1_ PC with _Network Sharing_ enabled. Unfortunately there was some challenges with the application.

## Troubleshooting
### Is it an application issue ?
When I attempted to use the application, my PC could not detect the 3DS through Windows Network Sharing. I initially suspected a Windows 10 compatibility issue but continued troubleshooting.

I then considered that the network protocol might be blocked or unsupported. A quick search showed that the microSD Management application uses _SMBv1_, which is disabled by default in Windows 10. I enabled _SMBv1_ through Windows Features and allowed it through Windows Firewall, but the 3DS still could not connect.

### Is it related to networking ?
I decided to test with an alternative router instead of using the router from my ISP to rule out possible network issues. I used my _Cisco Linksys Wireless-G 2.4 GHz_ for testing and it worked with no issues.

## Analysis
### Why did the router from my ISP fail ?
I captured the traffic in _Wireshark_ from both routers and compared them against one another in hopes of finding something meaningful. The only thing that stood out was the difference in the version of the IGMP protocol that was utilized. The successful connection used _IGMPv2_ while _IGMPv3_ failed.

[![Wireshark Capture](/assets/images/3DS_Analysis/WiresharkCapture3DS_2.png)](/assets/images/3DS_Analysis/WiresharkCapture3DS_2.png)

I reviewed the settings in my ISPs router and it turned out _IGMPv1_ and _IGMPv2_ is blocked while _IGMPv3_ is set as the default protocol. After altering the settings the application was functional.

[![Wireshark Capture](/assets/images/3DS_Analysis/3DS_Post_2.png){: width="83%"}](/assets/images/3DS_Analysis/3DS_Post_2.png)

## Explanation
IGMP (Internet Group Management Protocol) is used by routers to manage multicast communication. Here, it helps the computer and Nintendo 3DS discover and connect to each other over the network.

The 3DS uses _IGMPv2_, but the router rejected it and replied with _IGMPv3_, which the 3DS does not support. As a result, the connection failed.

The Nintendo 3DS utilized a protocol that is no longer considered the standard for multicast communication.

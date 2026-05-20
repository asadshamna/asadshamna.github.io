---
layout: post
title: TryHackMe Blueprint Machine Walkthrough
date: 2026-05-20
categories: [writeups, tryhackme]
---

#### [](#header-4)Introduction

In this walkthrough, I will be performing a full penetration test on the TryHackMe Blueprint machine. The goal of this machine is to enumerate the target, identify vulnerabilities, gain initial access, escalate privileges, and finally capture the root flag.

This machine was very interesting because it demonstrated how dangerous misconfigured web applications and exposed installation directories can be. Throughout this walkthrough, I will explain every step I performed during the exploitation process, along with some additional insights and explanations which can help beginners understand the methodology behind the attack chain.

* * * 

#### [](#header-4)Host Discovery

Before starting enumeration, I first verified whether the target machine was alive by sending ICMP packets using the ping command.

```js
ping -c 5 10.48.162.178
PING 10.48.162.178 (10.48.162.178) 56(84) bytes of data.
64 bytes from 10.48.162.178: icmp_seq=1 ttl=126 time=166 ms
64 bytes from 10.48.162.178: icmp_seq=2 ttl=126 time=64.6 ms
64 bytes from 10.48.162.178: icmp_seq=3 ttl=126 time=283 ms
64 bytes from 10.48.162.178: icmp_seq=4 ttl=126 time=239 ms
64 bytes from 10.48.162.178: icmp_seq=5 ttl=126 time=281 ms

--- 10.48.162.178 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 64.565/206.503/282.587/82.571 ms
```

The host responded successfully, which confirmed that the machine was online and reachable from my attacking machine.

* * * 

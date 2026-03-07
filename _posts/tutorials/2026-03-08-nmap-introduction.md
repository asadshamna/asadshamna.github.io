---
layout: post
title: Introduction to Nmap
date: 2026-03-08
categories: [tutorials, nmap]
---

[Home](/) → [Tutorials](/tutorials/) → [Nmap](/tutorials/#nmap)

---

Nmap is a powerful network scanning tool used by penetration testers for discovering hosts and services on a network.

It allows security professionals to identify open ports, running services, operating systems, and potential vulnerabilities within a network.

## Why Nmap is Important in Penetration Testing

Nmap is one of the most widely used tools in cybersecurity because it helps testers:

- Discover live hosts on a network
- Identify open ports
- Detect running services and versions
- Perform vulnerability detection using NSE scripts

## Basic Example

A simple Nmap scan can be performed using:
```js
nmap -sC -sV 192.168.1.10
```

This command performs:

- Default script scanning (`-sC`)
- Version detection (`-sV`)

## What You Will Learn in This Series

In this Nmap tutorial series we will cover:

- Nmap installation
- Basic scanning techniques
- Port scanning methods
- Service enumeration
- Nmap scripting engine (NSE)
- Advanced scanning techniques

---

Stay tuned for the next tutorial in the **Nmap series**.
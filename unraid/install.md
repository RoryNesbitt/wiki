---
title: Unraid installation 
description: Installation instructions for unraid
published: 1
date: 2021-06-20T21:20:45.098Z
tags: server, unraid
editor: markdown
dateCreated: 2021-06-18T21:20:09.447Z
---

## Preparation

### USB

Firstly get the most up to date version of the [flash creation tool](https://unraid.net/download) and choose a high quality USB device to install on to. A USB 2.0 is recommended as the speed is not relevant and they seem to last longer than 3.0. I used a cisco branded USB, mostly as it has a flashing blue light when in use, which indicates when the server is writing to it. This doesn't happen often as unraid is loaded in to RAM.
Choose the latest version and select the USB drive then start writing.

### Bios

The bios has to be changed to `boot from USB` before anything else. Particularly if you plan to directly pass an SSD through to a VM as this could be booted from before the USB.
In order to create that VM in the first place `virtualisation` has to be enabled in the BIOS too.
It is a good idea at this point to turn on `Wake-on-LAN` if the motherboard supports it
Save these settings and reboot.

> At this point the server can be accessed remotely by putting either <http://tower.local> or the servers IP address in to a browser.  
Replace tower with the servers name if you modified it during USB creation.
{.is-info}

### License

Select use trial key. A full license can be purchased through the unraid gui at a later point. This will give you 30 days however this can be increased to 60 by requesting an extension during this time.

---

## Storage

### Set up the array

The Array is where the main storage for unraid is. I barely make use of this.
An array pool can contain up to 30 drives, 28 storage and 2 parity. If using the unraid server as a NAS this gives a good number of benefits.

- Having only one or two drives as parity means you get a much higher percent of your storage to use as actual storage compared to raid.
- As long as the parity drive matches or exceeds the size of the largest storage drive then the rest of them do not have to match.
- Files are not stripped across drives like raid so if a drive fails and you do not have a parity drive the other drives will be unaffected.

I have two 1TB drives, one for storage and one for parity. Which is currently no better than a simple main and backup but can scale easily.

### Add a cache drive

Adding an SSD as a cache drive will speed up writes for the array, as well as reads for shares stored only on cache.

---

## Settings

> It is a good idea to add a password to root before doing anything further. Do this via the users tab at the top of the page.
{.is-danger}

### Shares

Shares are the main method of storing data on unraid. By default this will contain `appdata`, `domains`, `isos`, and `system`. the important column here is cache as it will tell you how each share is stored.

### Community Applications

### Docker

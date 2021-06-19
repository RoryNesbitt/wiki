---
title: Unraid installation 
description: Installation instructions for unraid
published: 1
date: 2021-06-19T23:24:22.100Z
tags: server, unraid
editor: markdown
dateCreated: 2021-06-18T21:20:09.447Z
---

## Preperation
### USB
Firstly get the most up to date version of the [flash creation tool](https://unraid.net/download) and choose a high quality USB device to install on to. A USB 2.0 is recommended as the speed is not relevant and they seem to last longer than 3.0. I used a cisco branded USB, mostly as it has a flashing blue light when in use, which indicates when the server is writting to it. This doesn't happen often as unraid is loaded in to RAM.
Choose the latest version and select the USB drive then start writting.
### Bios
The bios has to be changed to <kbd>boot from USB</kbd> before anythign else. Particularly if you plan to directly pass an SSD through to a VM as this could be booted from before the USB.
In order to create that VM in the first place <kbd>virtualisation</kbd> has to be enabled in the BIOS too.
It is a good idea at this point to turn on <kbd>Wake-on-LAN</kbd> if the motherboard supports it
Save these settings and reboot.
### License
Select use trial key. A full license can be purchased through the unraid gui at a later point. This will give you 30 days however this can be increased to 60 by requesting an extension during this time.

---
## Storage
### Set up the array
The Array is where the main storage for unraid is. I barely make use of this.
An array pool can contain up to 30 drives, 28 storage and 2 parity. If using the unraid server as a NAS this gives a massive number of benifits.
- Having only one or two drives as parity means you get a much higher percent of your storage to use as actual storage compared to raid.
- As long as the parity drive matches or excedes the size of the largest storage drive then the rest of them do not have to match.
- Files are not stripped accross drives like raid so if a drive fails and you do not have a parity drive the other drives will be unaffected.

### Add a cache drive


---
## settings stuff idk
### Shares

### Docker

### Community Applications
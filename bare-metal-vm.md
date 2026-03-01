---
title: Bare Metal VM
description: Hardware passthrough for your existing OS
published: true
date: 2026-03-01T19:42:41.546Z
tags: 
editor: markdown
dateCreated: 2026-03-01T15:06:41.871Z
---

> ## WIP
> This page is very much a work in progress. I'll make it make sense when I get the commands working
> {.is-warning}

## Setting up the bootloader



## Installing KVM

```s
sudo pacman -S libvirt virt-manager qemu-full dnsmasq dmidecode
sudo systemctl enable --now libvertd.service virtlogd.service
virsh list
sudo usermod -aG libvirt rory
sudo virst list
sudo systemctl enable --now virtqemud
sudo virsh list
sudo pacman -Syu
yay -S qemu-full qemu-img libvirt virt-install virt-manager virt-viewer
sudo virsh net-autostart default
```

## Setting up the initial VM

> The single gpu passthrough comes from [this guide](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/home). I have extracted the parts that I need so refer to the original guide if your setup is different
> {.is-info}
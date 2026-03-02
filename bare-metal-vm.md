---
title: Bare Metal VM
description: Hardware passthrough for your existing OS
published: true
date: 2026-03-02T11:10:21.257Z
tags: 
editor: markdown
dateCreated: 2026-03-01T15:06:41.871Z
---

> ## WIP
> This page is very much a work in progress. I'll make it make sense when I get the commands working
> {.is-warning}

## Setting up Virtualisation

### System Requirements

Firstly, enable virtualisation in bios. Then for intel cpus you need to add the kernel parameter `intel_iommu=on`.

For limine bootloader this needs to be added with
```sh
sudoedit /etc/default/limine
```
it should go at the end of the line `KERNEL_CMDLINE[default]+="...."`

For other bootloaders it will be a similar process, check [the arch wiki](https://wiki.archlinux.org/title/Kernel_parameters) for how to do it. Or for cachyOS and limine see more [here](https://wiki.cachyos.org/configuration/boot_manager_configuration/#limine)


### Installing KVM

Install all the required packages for KVM/qemu etc. On arch based distro's it looks like this:
```sh
sudo pacman -S qemu-full virt-manager virt-viewer dnsmasq bridge-utils libguestfs ebtables vde2 openbsd-netcat
# If you plan to run windows 11 VMs you will also need swtpm
sudo pacman -S swtpm
```
Join the relevant groups
These should work as one command but for some reason I have to do them separate
```sh
sudo usermod -aG libvirt $USER
sudo usermod -aG libvirt-qemu $USER
sudo usermod -aG kvm $USER
sudo usermod -aG qemu $USER
```
And then run the following to set up the system daemons and default network
```sh
sudo systemctl enable --now libvirtd
sudo systemctl enable --now virtqemud
sudo systemctl enable --now virtnetworkd
sudo systemctl enable --now virtstoraged
sudo virsh net-autostart default
sudo virsh net-start default
```
At this point you should be able to run virt-manager and create VMs

## Setting up the VM

Firstly, in virt-manager go to edit, prefrences and make sure 'Enable XML editing is selected'. If you forget you will have to do this before you can add the storage.

### The starter VM

This is not going to be bare-metal-like. This is just to get the original OS virtualised.

Since we are going to be using an existing OS we want virt-manager to do as little as possible.

This is not going to be 
- Select Manual install
- choose the correct OS. This is important for Windows 11 as it will add the secure boot version of the firmware
- Allocate the memory and CPUs you want. We can tweak this later
- Important! Unselect 'Enable storage for this VM'
- Enter the name you want and then click finish. We don't need to do any customisation yet.

The VM will be created but will not be able to boot since there's no bootable media. This is when we give it the old disk.
Shutdown the VM, in my experience you will need to force-off this time. Now open the hardware details and we are going to give it storage.

Click 'Add Hardware' and select storage, it should be the default anyway. then
select XML and copy in the following:

```xml
<disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
  <source dev="/dev/<selected-disk>"/>
  <target dev="sda" bus="sata"/>
  <address type="drive" controller="0" bus="0" target="0" unit="0"/>
</disk>
```

Make sure you change `<selected-disk>` to whichever drive you want to use. In my case I have a secondary SSD with windows on it so mine looks like `<source dev="/dev/<selected-disk>"/>`

> It would be a good idea to add [the virtio drivers](https://github.com/virtio-win/kvm-guest-drivers-windows/wiki/Driver-installation) at this point too. However it's been so long since I had to do this part I'm not comfortable giving a definitive method I haven't tested.
> {.is-warning}

> For windows 11 you will likely need TPM. It might not seem to cause issues at first but you know what windows is like.
> Click 'Add Hardware', select 'TPM'. Make sure the following are correct;
> Type: Emulated
> Model: CRB
> Version: 2.0
> {.is-warning}



You can now start your VM and you will be greated by your existing OS installation. I like to keep this version of it around for when I want to run something in the background without taking over my whole system.

### Adding GPU passthrough

> The single gpu passthrough comes from [this guide](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/home). I have extracted the parts that I need so refer to the original guide if your setup is different
> {.is-info}

We're going to use [risingprismtvs single-gpu-passthrough scripts](https://gitlab.com/risingprismtv/single-gpu-passthrough). This is the same repo as the guide linked to above. However I make a slight modification so that they will trigger for multiple VMs, not just the one called 'win10'.

```sh
git clone https://gitlab.com/risingprismtv/single-gpu-passthrough
cd single-gpu-passthrough
sed -i 's/"win10"/*"-gpu"/' hooks/qemu
sudo ./install_hooks.sh
cd ..
rm -rf single-gpu-passthrough
```

The change here lets the hook run for any VM that's name ends '-gpu'.
To confirm it worked check that these files exist.

```sh
/etc/systemd/system/libvirt-nosleep@.service
/usr/local/bin/vfio-startup
/usr/local/bin/vfio-teardown
/etc/libvirt/hooks/qemu
```

> If it is not clear which iommu group to pass through check out [this script](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/3)-IOMMU-Groups) from the original guide
> {.is-info}

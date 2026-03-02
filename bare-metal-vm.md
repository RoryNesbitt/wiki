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

# Setting up Virtualisation

## System Requirements

Firstly, enable virtualisation in bios. Then for intel cpus you need to add the kernel parameter `intel_iommu=on`.

For limine bootloader this needs to be added with
```sh
sudoedit /etc/default/limine
```
it should go at the end of the line `KERNEL_CMDLINE[default]+="...."`

For other bootloaders it will be a similar process, check [the arch wiki](https://wiki.archlinux.org/title/Kernel_parameters) for how to do it. Or for cachyOS and limine see more [here](https://wiki.cachyos.org/configuration/boot_manager_configuration/#limine)


## Installing KVM

Install all the required packages for KVM/qemu etc. On arch based distro's it looks like this:
```sh
sudo pacman -S qemu-full virt-manager virt-viewer dnsmasq bridge-utils libguestfs ebtables vde2 openbsd-netcat
 If you plan to run windows 11 VMs you will also need swtpm
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

# Setting up the VM

Firstly, in virt-manager go to edit, prefrences and make sure 'Enable XML editing is selected'. If you forget you will have to do this before you can add the storage.

## The starter VM

# tabs-gpu {.tabset}
## Guide

> There is something I've missed here that stops you from changing the resolution in a windows VM
> I'll update this when I figure it out. Until then, the complete xml example doesn't have that issue.
> {.is-danger}

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

You can now start your VM and you will be greeted by your existing OS installation. I like to keep this version of it around for when I want to run something in the background without taking over my whole system.

## XML
```xml
<domain type="kvm">
  <name>win11</name>
  <uuid>01654e6e-f4eb-48d5-a5f6-bcd9d7dcbbe0</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://microsoft.com/win/11"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="KiB">33554432</memory>
  <currentMemory unit="KiB">33554432</currentMemory>
  <vcpu placement="static">16</vcpu>
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-7.1">hvm</type>
    <firmware>
      <feature enabled="no" name="enrolled-keys"/>
      <feature enabled="yes" name="secure-boot"/>
    </firmware>
    <loader readonly="yes" secure="yes" type="pflash" format="raw">/usr/share/edk2/x64/OVMF_CODE.secboot.4m.fd</loader>
    <nvram template="/usr/share/edk2/x64/OVMF_VARS.4m.fd" templateFormat="raw" format="raw">/var/lib/libvirt/qemu/nvram/win11_VARS.fd</nvram>
    <boot dev="hd"/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
    </hyperv>
    <vmport state="off"/>
    <smm state="on"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <topology sockets="1" dies="1" clusters="1" cores="8" threads="2"/>
  </cpu>
  <clock offset="localtime">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type="block" device="disk">
      <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
      <source dev="/dev/sdb"/>
      <target dev="sda" bus="sata"/>
      <address type="drive" controller="0" bus="0" target="0" unit="0"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x02" slot="0x00" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pcie-root"/>
    <controller type="pci" index="1" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="1" port="0x10"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="2" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="2" port="0x11"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x1"/>
    </controller>
    <controller type="pci" index="3" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="3" port="0x12"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x2"/>
    </controller>
    <controller type="pci" index="4" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="4" port="0x13"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x3"/>
    </controller>
    <controller type="pci" index="5" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="5" port="0x14"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x4"/>
    </controller>
    <controller type="pci" index="6" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="6" port="0x15"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x5"/>
    </controller>
    <controller type="pci" index="7" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="7" port="0x16"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x6"/>
    </controller>
    <controller type="pci" index="8" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="8" port="0x17"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x7"/>
    </controller>
    <controller type="pci" index="9" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="9" port="0x18"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="10" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="10" port="0x19"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x1"/>
    </controller>
    <controller type="pci" index="11" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="11" port="0x1a"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x2"/>
    </controller>
    <controller type="pci" index="12" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="12" port="0x1b"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x3"/>
    </controller>
    <controller type="pci" index="13" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="13" port="0x1c"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x4"/>
    </controller>
    <controller type="pci" index="14" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="14" port="0x1d"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x5"/>
    </controller>
    <controller type="sata" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1f" function="0x2"/>
    </controller>
    <controller type="virtio-serial" index="0">
      <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
    </controller>
    <interface type="network">
      <mac address="52:54:00:a3:62:5b"/>
      <source network="default"/>
      <model type="e1000e"/>
      <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
    </interface>
    <serial type="pty">
      <target type="isa-serial" port="0">
        <model name="isa-serial"/>
      </target>
    </serial>
    <console type="pty">
      <target type="serial" port="0"/>
    </console>
    <channel type="spicevmc">
      <target type="virtio" name="com.redhat.spice.0"/>
      <address type="virtio-serial" controller="0" bus="0" port="1"/>
    </channel>
    <input type="tablet" bus="usb">
      <address type="usb" bus="0" port="1"/>
    </input>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <tpm model="tpm-crb">
      <backend type="emulator" version="2.0">
        <profile name="default-v1"/>
      </backend>
    </tpm>
    <graphics type="spice" autoport="yes">
      <listen type="address"/>
      <image compression="off"/>
    </graphics>
    <sound model="ich9">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1b" function="0x0"/>
    </sound>
    <audio id="1" type="spice"/>
    <video>
      <model type="virtio" heads="1" primary="yes"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
    </video>
    <redirdev bus="usb" type="spicevmc">
      <address type="usb" bus="0" port="2"/>
    </redirdev>
    <redirdev bus="usb" type="spicevmc">
      <address type="usb" bus="0" port="3"/>
    </redirdev>
    <watchdog model="itco" action="reset"/>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </memballoon>
  </devices>
</domain>

```

## Adding GPU Passthrough Capability


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

## Creating a VM with passthrough

# Tabs {.tabset}
## Guide

## XML

```xml
<domain type="kvm">
  <name>win11-gpu</name>
  <uuid>d0be9ec3-5a15-46c9-9da8-ce4c1dfc2bfb</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://microsoft.com/win/10"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="KiB">33554432</memory>
  <currentMemory unit="KiB">33554432</currentMemory>
  <vcpu placement="static">16</vcpu>
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-7.1">hvm</type>
    <firmware>
      <feature enabled="no" name="enrolled-keys"/>
      <feature enabled="yes" name="secure-boot"/>
    </firmware>
    <loader readonly="yes" secure="yes" type="pflash" format="raw">/usr/share/edk2/x64/OVMF_CODE.secboot.4m.fd</loader>
    <nvram template="/usr/share/edk2/x64/OVMF_VARS.4m.fd" templateFormat="raw" format="raw">/var/lib/libvirt/qemu/nvram/win11-gpu_VARS.fd</nvram>
    <boot dev="hd"/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
    </hyperv>
    <vmport state="off"/>
    <smm state="on"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <topology sockets="1" dies="1" clusters="1" cores="8" threads="2"/>
  </cpu>
  <clock offset="localtime">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type="block" device="disk">
      <driver name="qemu" type="raw" cache="none" io="native" discard="unmap"/>
      <source dev="/dev/sdb"/>
      <target dev="sda" bus="sata"/>
      <address type="drive" controller="0" bus="0" target="0" unit="0"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x02" slot="0x00" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pcie-root"/>
    <controller type="pci" index="1" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="1" port="0x10"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="2" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="2" port="0x11"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x1"/>
    </controller>
    <controller type="pci" index="3" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="3" port="0x12"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x2"/>
    </controller>
    <controller type="pci" index="4" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="4" port="0x13"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x3"/>
    </controller>
    <controller type="pci" index="5" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="5" port="0x14"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x4"/>
    </controller>
    <controller type="pci" index="6" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="6" port="0x15"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x5"/>
    </controller>
    <controller type="pci" index="7" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="7" port="0x16"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x6"/>
    </controller>
    <controller type="pci" index="8" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="8" port="0x17"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x7"/>
    </controller>
    <controller type="pci" index="9" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="9" port="0x18"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="10" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="10" port="0x19"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x1"/>
    </controller>
    <controller type="pci" index="11" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="11" port="0x1a"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x2"/>
    </controller>
    <controller type="pci" index="12" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="12" port="0x1b"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x3"/>
    </controller>
    <controller type="pci" index="13" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="13" port="0x1c"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x4"/>
    </controller>
    <controller type="pci" index="14" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="14" port="0x1d"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x5"/>
    </controller>
    <controller type="sata" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1f" function="0x2"/>
    </controller>
    <interface type="network">
      <mac address="52:54:00:42:03:87"/>
      <source network="default"/>
      <model type="e1000e"/>
      <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
    </interface>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <tpm model="tpm-crb">
      <backend type="emulator" version="2.0">
        <profile name="default-v1"/>
      </backend>
    </tpm>
    <audio id="1" type="none"/>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
      </source>
      <rom file="/usr/share/vgabios/gpu.rom"/>
      <address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>
    </hostdev>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>
      </source>
      <rom file="/usr/share/vgabios/gpu.rom"/>
      <address type="pci" domain="0x0000" bus="0x06" slot="0x00" function="0x0"/>
    </hostdev>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x01" slot="0x00" function="0x2"/>
      </source>
      <rom file="/usr/share/vgabios/gpu.rom"/>
      <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>
    </hostdev>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>
      </source>
      <rom file="/usr/share/vgabios/gpu.rom"/>
      <address type="pci" domain="0x0000" bus="0x08" slot="0x00" function="0x0"/>
    </hostdev>
    <hostdev mode="subsystem" type="pci" managed="yes">
      <source>
        <address domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
      </source>
      <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
    </hostdev>
    <watchdog model="itco" action="reset"/>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </memballoon>
  </devices>
</domain>
```

> If it is not clear which iommu group to pass through check out [this script](https://gitlab.com/risingprismtv/single-gpu-passthrough/-/wikis/3)-IOMMU-Groups) from the original guide
> {.is-info}

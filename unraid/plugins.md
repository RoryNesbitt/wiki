---
title: Plugins
description: A short list of useful plugins
published: 1
date: 2021-07-07T16:21:27.507Z
tags: organisation
editor: markdown
dateCreated: 2021-06-23T15:04:59.432Z
---

# Community Applications

# tab {.tabset}

## Overview

> Community applications is the appstore for unraid. it is the only plugin or container that you should ever need to manually install.
{.is-info}

Installing plugins or docker containers on unraid is mainly done through the CA apps tab. Once installed this tab provides a searchable grid layout for available addons. These are not managed by CA but rather any unraid user can request their repositories be added.  
Community Applications was created by the user Squid. Squid has also created many other plugins and you would not go wrong selecting his repo and installing every one of them, although some of the most useful ones will be detailed bellow.

## installation

To add in CA to your server you go to the **plugins** tab, click on **Install plugin** and then copy this URL in to the text box shown:

```txt
https://raw.githubusercontent.com/Squidly271/community.applications/master/plugins/community.applications.plg
```

Click install and let this run, it should go quick, and then refresh the page to see the apps tab now available.

# Essential plugins

# tab {.tabset}

## Appdata Backup/Restore v2

### Squid's Repository

This plugin will create a backup of your appdata share and store it somewhere better protected such as the parity protected array or a network drive. It is also able to run this on specific days and times for scheduled backups.  
The backups can then be restored from within this same plugin.
>This should be done when the server is not being used as it will take all docker containers offline while it runs.
{.is-warning}

## Dynamix SSD Trim

### Dynamix Repository

This plugin is highly recommended if you are using an xfs formatted SSD for your cache drive.  
This plugin will mark deleted pages on your SSD as Stale meanign when the controller is copying stored data to a new block it will nmto also copy over the deleted pages. Over time this will have a large effect ont he number of writes to each page which should extend teh SSD's lifespan.

## Fix Common Problems

### Squid's Repository

Fix Common Problems does exactly what you expect. It will periodically scan your server and notify you of any common unraid issues which it notices. This will work out of the box and it will also suggest simple fixes to most of the errors it finds. Probably the most important plugin for any unraid server.

## My Servers

### Official Unraid Repository

This is an official unraid plugin which will connect to your unraid.net account and display server information on unraid.net as well as account information on your own server. It also alows for remote access to your server, although in my experience this is a bit finicky if you ever decide to change it's setup.

The reason this plugin makes it in to essential however is it's ability to backup your USB drive to your unraid.net account. This will make the usual zip backup which can be downloaded through the gui and instead store it off site meaning at any point you can log in to your account and get an up to data copy of all your settings for USB recovery.

## NerdPack GUI

### Dmacias' Repository

Nerdpack allows you to select extra CLI packages to download and install. This is essential for anyone wanting to SSH in to their server as any packages tehy add manually will get wiped when they reboot and unraid is loaded from USB again.  
Combining this with a userscript I install zsh and retrieve my configs from github so my CLI will match my other linux devices.

## Preclear Disk

### Gfjardim's Repository

Preclear Disks gives an informative GUI to a couple preclear scripts which allow you to 'zero' new hard disks before you add them to your array.  
This is a proccess that takes several hours and by doing it with this plugin it can be run with the array still active saving many hours of downtime. It has the options to run multiple times (3 is recommended) and to perform a full read operation on every bit before and after each run to check for any errors in the drive.

> A docker container, binhex preclear, is also availible to do the same task. Binhex is a common name that will pop up when searching containers but I have never used this one so I can't recommend it over gfjardim's plugin.
{.is-info}

> This plugin still states that you must install the scripts seperate but the support page will confirm that it now includes Gfjardim's self made script as well as a copy of bjp999's script.
{.is-info}

## Unassigned Devices (Plus)

### Dlandon's Repository

Unassigned devices allows you to mount and access hard drives that have not been added to the array, useful for transferring files before preclearing.

Unassigned Devices plus add access to additional formats and a destructive mode which lets you erase and format the drives from this plugin.

## User Scripts

### Squid's Repository

Users Scripts is the unraid version of crontab. It allows you to write your own custom scripts through the webgui and run them on a schedule or have them triggered by the starting and stopping of the array. It includes by default scripts to remove unussed docker images, cleanup .DS_Store files leaft by Apple's Finder and one to display the size of your docker log. I periodically run the docker image removal manually but I have no need so far for either of the other two.

Combining this with the rclone plugin I have a script to mount my cloud storage when the array starts and then unmount it when it stops.

---
title: Plugins
description: A short list of useful plugins
published: 1
date: 2021-07-03T20:03:44.478Z
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
{.is-info}

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

## Nerdpack

### Dmacias' Repository

## Preclear Disks

### Gfjardim's Repository

> A docker container, binhex preclear, is also availible to do the same task. Binhex is a common name that will pop up when searching containers but I have never used this one so I can't recommend it over gfjardim's plugin.
{.is-warning}

## Unassigned Devices (Plus)

### Dlandon's Repository

## User Scripts

### Squid's Repository

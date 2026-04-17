# TrueNAS Setup

## Overview

This document summarizes the core TrueNAS setup for my homelab environment, including storage configuration, network approach, and the reasoning behind key decisions.

## Initial Setup

Installed TrueNAS Community Edition on a dedicated 1TB NVMe boot drive, separate from the storage pool. The boot drive is larger than necessary for TrueNAS, but it was what I had available at the time. Keeping the OS drive independent from the data pool ensures that a storage drive failure doesn't take down the operating system.

## Storage Configuration

### Pool Design
- **Pool name:** `tank`
- **Layout:** RAIDZ1
- **Drives:** 3 × 20TB IronWolf Pro
- **Usable design goal:** balance of high-capacity storage with single-drive fault tolerance

### Why RAIDZ1
I chose RAIDZ1 as a balance between usable capacity and redundancy. The main goal of this build was to support large media storage and self-hosted services while still maintaining protection against a single drive failure.

I understand RAID is not a backup, but for this use case RAIDZ1 provided the best tradeoff between efficiency and resilience.

## Dataset / Folder Structure

The lab is organized around separate datasets for media, downloads, and application data. The structure was planned before deploying services to ensure clean path mapping into containerized applications and avoid conflicts between services sharing the same storage.

```
tank/
├── media/
│   ├── movies/
│   └── tv/
├── downloads/
│   ├── complete/
│   └── incomplete/
├── immich/
│   ├── library/
│   └── postgres/
└── apps/
```

One of the main goals in organizing storage this way was to avoid confusion when mapping host paths into containerized applications. Getting this right early prevented path-related issues that are common when multiple services need to access the same files.

## Network Configuration

After installation, I decided to use a **DHCP reservation** for the server instead of manually assigning a static IP. The reservation is configured on the TP-Link router, which keeps IP management centralized at the router level rather than on individual devices.

That decision simplified access to the web interface and reduced the risk of address conflicts during network changes.

## Goals of the Platform

This TrueNAS system serves as:
- centralized storage
- platform for containerized services
- hands-on lab for learning system administration
- environment for troubleshooting storage, networking, and service deployment issues

## Lessons Learned

- Keeping the boot drive separate from the storage pool protects the OS from data drive failures
- Stable addressing is important for infrastructure systems
- Storage layout decisions affect app configuration later
- Service deployment gets easier when folders and datasets are planned in advance
- It is worth documenting path structure early to prevent avoidable app issues later
- ZFS ACL type matters — NFSv4 ACLs can silently block container writes even when Unix permissions look correct

# Homelab Infrastructure Documentation

This repository documents my personal homelab, built for hands-on practice with storage, networking, remote access, containerized services, and system administration.

The environment is centered around a self-hosted TrueNAS server with ZFS RAIDZ1 storage, remote access through Tailscale and Cloudflare Tunnel, and containerized services used to learn service deployment, DNS troubleshooting, storage management, and network configuration.

## Purpose

I built this lab to gain practical, hands-on experience with infrastructure and networking by running real services in a self-hosted environment and troubleshooting problems as they came up.

Rather than treating the lab as just a media server, I use it as a platform for learning:

- TrueNAS administration
- ZFS storage management
- Remote access and secure connectivity
- Containerized service deployment
- DNS and network troubleshooting
- Service mapping and file path consistency
- Documentation of issues, fixes, and lessons learned

## Current Environment

### Core Platform
- **OS:** TrueNAS Community Edition (formerly SCALE) — Linux-based (Debian)
- **Storage:** ZFS RAIDZ1
- **Pool:** `tank`
- **Drive Layout:** 3 × 20TB IronWolf Pro drives (~40TB usable after parity)
- **Primary Use Cases:** centralized storage, media services, infrastructure learning, remote access, containerized applications

### Networking and Remote Access
- DHCP reservation used for stable server addressing
- Secure remote access through **Tailscale**
- Additional external access and service publishing through **Cloudflare Tunnel**
- Local network built behind AT&T Fiber gateway and TP-Link router

### Services and Applications
- **Emby** — media server for streaming to local and remote devices
- **Radarr** — automated media library management (movies)
- **Sonarr** — automated media library management (TV)
- **NZBGet** — download client for automated services
- **Jellyseerr** — media request and discovery portal
- **Tailscale** — mesh VPN for secure remote access
- **Cloudflare Tunnel** — reverse tunnel for external service access without port forwarding
- **Immich** — self-hosted photo and video management platform for automatic mobile backups and private cloud access


## What This Repository Covers

This documentation focuses on:
- how the lab is structured
- why major design decisions were made
- problems encountered during setup
- how those problems were diagnosed and fixed
- lessons learned that apply to systems, networking, and infrastructure work

## Documentation Index

- [TrueNAS Setup](docs/truenas-setup.md)
- [Remote Access](docs/remote-access.md)
- [Troubleshooting Notes](docs/troubleshooting-notes.md)

## Key Lessons So Far

- DNS issues can break services in ways that are not immediately obvious
- Consistent container path mapping is critical for apps that rely on shared downloads and media folders
- Stable IP planning matters, and DHCP reservation can be cleaner than fully static addressing in a home environment
- Remote access should be designed intentionally instead of exposing services directly
- Troubleshooting works best when isolating one variable at a time
- Layered troubleshooting matters when problem solving multiple overlapping issues

## In Progress

Planned future additions:
- hardware overview
- network diagram
- storage layout documentation
- service-by-service application notes
- cybersecurity lab documentation for Kali Linux
- mini-rack network upgrade to expand the lab with OPNsense routing, Pi-hole and Unbound for DNS services, a Ubiquiti 10Gb managed switch for improved local transfer speeds, and a UniFi access point for wireless coverage

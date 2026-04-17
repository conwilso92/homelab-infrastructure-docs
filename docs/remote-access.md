# Remote Access

## Overview

This document covers how remote access is handled in my homelab and why I chose to use secure overlay and tunnel-based access rather than exposing services directly.

## Network Diagram

![Network Diagram](images/network-diagrams.svg)

## Design Decisions

No ports are forwarded on the router. All remote access goes through encrypted overlays or tunnels, reducing the attack surface of the homelab. This was an intentional choice — I wanted to learn how to provide remote connectivity without exposing internal infrastructure directly to the internet.

## Approach

I use a combination of:
- **Tailscale** for private remote access and administration
- **Cloudflare Tunnel** for controlled external access to selected services

Each tool has a distinct role. Tailscale is for my personal admin access — reaching the TrueNAS web interface, SSH, and internal services from anywhere. Cloudflare Tunnel is for sharing specific services with approved users without requiring them to install anything or join my network.

## Tailscale

Tailscale provides secure, mesh VPN access to my homelab without relying on traditional port forwarding. I can reach the TrueNAS web interface, SSH into the server, and access containerized services from my phone or laptop regardless of where I am.

### Why I Used It
- Secure remote access without opening any ports on the router
- Device-to-device connectivity across multiple locations
- Practical hands-on experience with VPN-style overlay networking
- Useful for testing service reachability from outside the local network

## Cloudflare Tunnel

Cloudflare Tunnel runs as a container on TrueNAS and provides external access to selected services through a custom domain. This allows approved users to reach specific services through a browser without needing VPN software or direct access to my network.

### Why I Used It
- Avoid directly exposing internal infrastructure to the internet
- Simplify remote connectivity for selected services and approved users
- Gain hands-on experience with modern tunnel-based access and reverse proxy concepts
- Learn more about DNS, routing, and externally reachable service publishing

## How They Work Together

Tailscale and Cloudflare Tunnel serve different purposes and are used side by side:

- **Tailscale** handles private admin access — I use it to manage TrueNAS, troubleshoot services, and reach anything on my network remotely. Only my authorized devices are on the tailnet.
- **Cloudflare Tunnel** handles external service access — specific services are published through a custom domain so approved users can access them from a browser without joining my network or installing any software.

This separation keeps administrative access isolated from service access, which reduces risk and makes troubleshooting cleaner.

## Lessons Learned

- Remote access should be designed intentionally, not added as an afterthought
- VPN-based access is often cleaner and safer than exposing services directly
- Keeping admin access and service access on separate paths simplifies security and troubleshooting
- Tunnel-based access adds convenience, but also adds another layer to troubleshoot when things break
- DNS problems can appear as app or service failures even when the real issue is upstream networking

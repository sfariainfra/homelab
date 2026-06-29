# Infrastructure Architecture

## Overview

The homelab is composed of three primary environments.

- Fedora Workstation
- Fedora Home Server
- Ubuntu VPS

Each machine has a specific role to simulate a small production environment.

---

## Fedora Workstation

Purpose:

- Daily Linux workstation
- Development
- Virt-Manager
- SSH management
- Neovim
- Testing

Responsibilities

- Administration
- Infrastructure management
- Remote access
- Development

---

## Fedora Home Server

Purpose

Main infrastructure server.

Responsibilities

- Docker
- Samba
- Jellyfin
- Calibre-Web
- Grafana
- Prometheus
- Loki
- Alloy
- Nginx
- dnsmasq
- Libvirt
- Virtual Machines

---

## Ubuntu VPS

Purpose

Public-facing services.

Responsibilities

- Football Hacking
- Django blog
- Nginx
- HTTPS
- Reverse proxy

---

## Network

Internet

↓

Ubuntu VPS

↓

Tailscale

↓

Fedora Home Server

↓

Docker + Virtual Machines

↓

Fedora Workstation

---

## Security

- SSH keys only
- Firewalld
- SELinux Enforcing
- HTTPS
- Tailscale
- Reverse Proxy

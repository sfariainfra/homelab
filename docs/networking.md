# Networking

## Overview

Networking is one of the most important parts of this homelab. The environment includes LAN services, Docker networks, libvirt NAT networks, Tailscale remote access, Nginx reverse proxying, firewalld zones, and public access through a VPS.

The main goal is to understand how traffic moves between machines, containers, virtual machines, local services, and public services.

## Network Layers

The environment can be viewed in layers:

```text
Internet
   |
   v
VPS public network
   |
   v
Tailscale private network
   |
   v
Home LAN
   |
   v
Docker networks / libvirt networks
```

## LAN

The local network hosts the Fedora Workstation and Fedora Home Server.

The home server provides private services such as:

- Jellyfin
- Calibre-Web
- Samba
- Grafana
- Prometheus
- Loki
- local Nginx reverse proxy
- virtual machines through libvirt

## Tailscale

Tailscale is used as a private mesh network between the main machines.

Main use cases:

- private remote access
- SSH access without exposing services directly to the internet
- administrative access to the home server
- safer access to internal services
- connection between workstation, home server, and VPS

## DNS

Local DNS is used to make internal services easier to access.

Examples:

```text
jellyfin.home
grafana.home
books.home
```

Instead of accessing services only by IP address and port, local names make the environment cleaner and closer to a real infrastructure setup.

## Nginx Reverse Proxy

Nginx is used both locally and publicly.

Main responsibilities:

- route traffic to internal services
- terminate HTTPS on public services
- hide backend ports from direct user access
- provide stable service URLs
- separate public and private exposure

Example flow:

```text
User
 |
 v
Nginx
 |
 v
Backend service
```

## Firewalld

Firewalld is used as the main host firewall.

It controls which ports and services are accessible from each network zone.

Important concepts used in this homelab:

- zones
- services
- ports
- interfaces
- runtime vs permanent rules
- Docker interaction with nftables
- libvirt interaction with NAT networks

## Docker Networking

Docker creates its own networking rules and NAT behavior.

Important notes:

- Docker can publish ports even when a service is not explicitly opened in firewalld.
- Published ports may appear in nftables rules created by Docker.
- This can cause confusion when troubleshooting access.
- It is important to check both firewalld and Docker-generated rules.

Useful commands:

```bash
docker ps
docker network ls
sudo nft list ruleset
sudo firewall-cmd --list-all
```

## Libvirt Networking

Libvirt uses virtual networks for virtual machines.

Common components:

- virbr0
- NAT
- DHCP
- dnsmasq
- forwarding
- masquerading

Typical troubleshooting areas:

- VM gets an IP but cannot reach the internet
- forwarding rules missing
- masquerade not applied
- firewalld zone not allowing forwarding
- default network inactive

Useful commands:

```bash
virsh net-list --all
virsh net-info default
ip addr show virbr0
sudo sysctl net.ipv4.ip_forward
sudo nft list ruleset
```

## SSH

SSH is the main administration protocol.

Use cases:

- connecting to the home server
- connecting to the VPS
- using SSH jump hosts
- managing remote libvirt with Virt-Manager
- opening temporary tunnels for database synchronization

## SSH Jump Host

Jump hosts are useful when a target machine is not directly reachable.

Example concept:

```text
Workstation
   |
   v
Jump host
   |
   v
Target machine
```

## Common Networking Lessons

This homelab has produced several practical networking lessons:

- If a service is running but not reachable, check the bind address.
- `localhost` may resolve to IPv4 or IPv6 depending on the system.
- Nginx may try IPv6 first when proxying to `localhost`.
- Docker can create firewall rules outside the obvious firewalld configuration.
- A VM can receive DHCP and still fail to access the internet.
- NAT requires forwarding and masquerading.
- DNS problems can look like application problems.
- Firewalls should be inspected before blaming the application.

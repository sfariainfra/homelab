# Services

## Overview

This document lists the main services running in the homelab and their purpose. The goal is to keep a clear inventory of what exists, where it runs, and why it exists.

## Service Inventory

| Service | Purpose | Host | Type |
|---|---|---|---|
| Jellyfin | Media server | Home Server | Docker |
| Calibre-Web | Ebook library | Home Server | Docker |
| Samba | File sharing | Home Server | Native |
| Grafana | Dashboards | Home Server | Native / service |
| Prometheus | Metrics collection | Home Server | Native / service |
| Loki | Log aggregation | Home Server | Docker |
| Alloy | Metrics and logs collector | Home Server / VPS | Docker |
| Nginx | Reverse proxy | Home Server / VPS | Native |
| n8n | Automation workflows | Home Server | Docker |
| Node Exporter | Host metrics | All hosts | Native / binary |
| Tailscale | Private mesh VPN | All hosts | Native |
| KVM/libvirt | Virtualization | Home Server / Workstation | Native |

## Jellyfin

Purpose:

Jellyfin is used as the private media server.

Host:

```text
Fedora Home Server
```

Type:

```text
Docker container
```

Main networking point:

```text
Port 8096/tcp
```

Relevant lesson:

A service can be healthy inside Docker and still be unreachable from the LAN if host firewall rules do not allow the published port.

## Calibre-Web

Purpose:

Calibre-Web is used as an ebook library and web interface for books.

Host:

```text
Fedora Home Server
```

Type:

```text
Docker container
```

Main networking point:

```text
Port 8083/tcp
```

Relevant lesson:

Docker port publishing and firewalld rules must be understood together. A Docker service may be reachable because Docker added nftables rules even when no explicit firewalld rule was added.

## Samba

Purpose:

Samba provides local file sharing.

Use cases:

- shared media directories
- shared documents
- Obsidian/knowledge base storage
- access from workstation to server files

Host:

```text
Fedora Home Server
```

Security considerations:

- restrict access to trusted LAN clients
- avoid exposing Samba to the public internet
- use proper Linux permissions
- keep SELinux contexts correct

## Grafana

Purpose:

Grafana is used for dashboards and infrastructure visualization.

Host:

```text
Fedora Home Server
```

Data sources:

- Prometheus
- Loki

Dashboards:

- host metrics
- service metrics
- logs
- infrastructure overview

## Prometheus

Purpose:

Prometheus collects metrics from hosts and services.

Host:

```text
Fedora Home Server
```

Targets:

- Fedora Workstation
- Fedora Home Server
- Ubuntu VPS
- Node Exporter endpoints
- other exporters when added

## Loki

Purpose:

Loki is used for log aggregation.

Host:

```text
Fedora Home Server
```

Type:

```text
Docker container
```

Used with:

- Alloy
- Grafana

## Alloy

Purpose:

Alloy is used to collect and ship logs and metrics.

Hosts:

- Home Server
- VPS

Used with:

- Loki
- Prometheus
- Grafana

## Nginx

Purpose:

Nginx acts as a reverse proxy.

Hosts:

- Home Server
- VPS

Use cases:

- public application routing
- local service routing
- HTTPS termination
- proxying traffic to backend services
- separating service URLs from backend ports

## n8n

Purpose:

n8n is used for automation experiments and workflow orchestration.

Host:

```text
Fedora Home Server
```

Type:

```text
Docker container
```

## Node Exporter

Purpose:

Node Exporter exposes host-level metrics.

Hosts:

- Fedora Workstation
- Fedora Home Server
- Ubuntu VPS

Metrics collected:

- CPU
- memory
- disk
- filesystem
- network
- system load

## Tailscale

Purpose:

Tailscale provides private remote access across machines.

Hosts:

- Workstation
- Home Server
- VPS

Use cases:

- private SSH
- remote administration
- secure access to internal services
- avoiding unnecessary public exposure

## KVM / Libvirt

Purpose:

KVM and libvirt are used for virtual machines and isolated Linux labs.

Hosts:

- Fedora Home Server
- Fedora Workstation

Use cases:

- Rocky Linux VMs
- system administration practice
- networking experiments
- safe troubleshooting labs

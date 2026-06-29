# Monitoring

## Overview

Monitoring is implemented to provide visibility into the homelab environment. The stack is based on Prometheus, Grafana, Loki, Alloy, and Node Exporter.

The goal is to observe hosts, containers, logs, services, and infrastructure behavior instead of troubleshooting blindly.

## Stack

```text
Node Exporter
     |
     v
Prometheus
     |
     v
Grafana
```

For logs:

```text
System / Service logs
     |
     v
Alloy
     |
     v
Loki
     |
     v
Grafana
```

## Prometheus

Prometheus is responsible for collecting metrics.

Typical targets:

- Fedora Workstation
- Fedora Home Server
- Ubuntu VPS
- Node Exporter endpoints
- service exporters when available

Main use cases:

- host health
- CPU usage
- memory usage
- disk usage
- filesystem monitoring
- network traffic
- service availability

## Node Exporter

Node Exporter exposes host metrics for Prometheus.

Installed on:

- Workstation
- Home Server
- VPS

Useful metric categories:

- CPU
- memory
- disk
- filesystem
- network
- system load
- kernel statistics

## Grafana

Grafana is used to visualize metrics and logs.

Main dashboards:

- host overview
- CPU and memory
- disk usage
- service health
- network traffic
- logs through Loki

## Loki

Loki is used for log aggregation.

Purpose:

- centralize logs
- search service logs
- inspect errors
- correlate logs with metrics
- troubleshoot incidents

## Alloy

Alloy is used to collect logs and send them to Loki.

Hosts:

- Home Server
- VPS

Main role:

- read logs
- label log streams
- forward logs to Loki
- support observability across machines

## Why Monitoring Matters

Monitoring helps answer practical questions:

- Is the server overloaded?
- Is disk usage growing?
- Did a service restart?
- Are logs showing errors?
- Is the VPS reachable?
- Is a problem related to the app, the network, or the host?
- Did a firewall change affect connectivity?

## Troubleshooting Workflow

A typical troubleshooting workflow:

1. Check whether the service is running.
2. Check whether the port is listening.
3. Check whether the firewall allows traffic.
4. Check whether Nginx can reach the backend.
5. Check logs in Loki or directly with journalctl.
6. Check host metrics in Grafana.
7. Confirm the fix from the client side.

Useful commands:

```bash
systemctl status <service>
journalctl -u <service> -xe
ss -lntp
docker ps
docker logs <container>
sudo firewall-cmd --list-all
curl -I http://localhost:<port>
```

## Future Improvements

Planned improvements:

- alerting rules
- uptime checks
- service-level dashboards
- container metrics
- blackbox exporter
- automated incident notes

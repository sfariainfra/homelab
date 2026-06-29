# Troubleshooting Notes

## Overview

This document records real troubleshooting scenarios from the homelab. The goal is to document not only the final fix, but the investigation logic used to reach it.

This is one of the most important parts of the repository because it shows practical Linux administration: observing symptoms, forming hypotheses, checking evidence, applying fixes, and validating results.

---

# Jellyfin Inaccessible from LAN

## Problem

Jellyfin was running in Docker, but it was not accessible from the local network.

## Symptoms

- Container appeared to be running.
- Service was expected on port `8096/tcp`.
- Access from the LAN failed.
- The problem looked like an application issue at first.

## Investigation

The first step was to verify whether the container was running.

```bash
docker ps
```

Then the listening ports and firewall were checked.

```bash
ss -lntp
sudo firewall-cmd --list-all
sudo nft list ruleset | grep 8096
```

The investigation showed that firewall behavior was involved.

## Cause

The service port was not allowed properly through the host firewall.

## Solution

Open the Jellyfin port in firewalld.

```bash
sudo firewall-cmd --add-port=8096/tcp --permanent
sudo firewall-cmd --reload
```

## Validation

After opening the port, access to Jellyfin from the LAN worked immediately.

## Lesson Learned

When a Docker service is running but unreachable, do not assume the application is broken. Check:

- container status
- published ports
- host firewall
- nftables rules
- service bind address

---

# Docker and Firewalld Interaction

## Problem

Some Docker services appeared reachable even without explicit firewalld rules, while others required firewall changes.

## Symptoms

- One container could be reached through its published port.
- Another container needed a firewalld port rule.
- The behavior looked inconsistent.

## Investigation

Docker-generated nftables rules were inspected.

```bash
sudo nft list ruleset
```

Docker port publishing was checked.

```bash
docker ps
```

Firewalld configuration was checked.

```bash
sudo firewall-cmd --list-all
```

## Cause

Docker can create its own NAT and forwarding rules. This means Docker networking and firewalld do not always behave as a single obvious layer.

## Lesson Learned

When troubleshooting Docker access on a firewalld-based system, inspect both:

```bash
sudo firewall-cmd --list-all
sudo nft list ruleset
```

Docker networking is not separate from host networking. It modifies the packet path.

---

# Nginx Proxy to Streamlit Failing

## Problem

A Streamlit application behind Nginx returned errors because Nginx could not connect to the upstream service.

## Symptoms

Nginx error logs showed connection refused errors when proxying requests to the Streamlit backend.

Typical log pattern:

```text
connect() failed (111: Connection refused) while connecting to upstream
```

## Investigation

Nginx configuration referenced:

```nginx
proxy_pass http://localhost:8501;
```

The backend listening address needed to be checked.

Useful commands:

```bash
sudo grep -R "localhost:8501" /etc/nginx
ss -lntp | grep 8501
sudo systemctl status <streamlit-service>
sudo journalctl -u <streamlit-service> -xe
```

## Cause

The backend and proxy were not aligned correctly. One important detail was the behavior of `localhost`, which can resolve to IPv4 or IPv6 depending on the system.

If Nginx tries `::1` while the app only listens on `127.0.0.1`, the proxy can fail.

## Solution

Use an explicit IPv4 loopback address when the backend listens only on IPv4:

```nginx
proxy_pass http://127.0.0.1:8501;
```

Then reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Lesson Learned

`localhost` is not always the same as `127.0.0.1`.

When proxying local services, be explicit if the backend only listens on IPv4.

---

# Libvirt VM Gets IP but Has No Internet

## Problem

A virtual machine received an IP address from libvirt DHCP but could not reach the internet.

## Symptoms

- VM received an IP address.
- VM could see its local network configuration.
- External ping failed.
- DNS was not the first suspected cause because even IP-based connectivity failed.

## Investigation

Inside the VM:

```bash
ip addr
ip route
ping 8.8.8.8
```

On the host:

```bash
virsh net-list --all
virsh net-info default
ip addr show virbr0
sudo sysctl net.ipv4.ip_forward
sudo nft list ruleset
sudo firewall-cmd --list-all
```

## Cause

The VM network had DHCP, but NAT/forwarding/masquerading rules were not correctly allowing outbound connectivity.

## Solution

Ensure the libvirt network is active and forwarding/masquerade rules are correct.

Useful checks:

```bash
virsh net-start default
virsh net-autostart default
sudo firewall-cmd --zone=libvirt --list-all
```

If needed, add forwarding and masquerade rules according to the host networking setup.

## Lesson Learned

DHCP success only proves that the VM reached libvirt DHCP. It does not prove that NAT, forwarding, DNS, or upstream routing is working.

---

# SSH Jump Host Argument Error

## Problem

An SSH alias using `-J` returned an invalid jump host argument error.

## Symptoms

Running an alias like this failed:

```bash
ssh -J sfaria:homeserver user@target
```

## Cause

The `-J` argument expects a valid jump host specification. A colon is interpreted as part of the SSH jump syntax for host and port, not as a separator between two host aliases.

## Correct Concept

Use one jump host at a time, or chain them correctly through SSH config.

Example:

```bash
ssh -J jumpuser@jumphost targetuser@targethost
```

For more complex cases, prefer `~/.ssh/config`.

## Lesson Learned

SSH aliases are useful, but once jump hosts become more complex, `~/.ssh/config` is cleaner, safer, and easier to debug.

---

# SSH Tunnel Synchronization

## Problem

Database synchronization through SSH tunnels could hang or become confusing when multiple tunnels were open at the same time.

## Symptoms

- terminal appeared stuck
- SSH agent CPU usage increased
- tunnel state was unclear
- multiple destinations were involved

## Investigation

The synchronization flow was reviewed.

The safer approach was to make the process sequential:

1. open tunnel to one target
2. run synchronization
3. close tunnel
4. open tunnel to next target
5. run synchronization
6. close tunnel

## Lesson Learned

For scripts that use SSH tunnels, predictable lifecycle management is critical.

A good tunnel script should:

- open only what is needed
- validate the tunnel
- run the task
- close the tunnel
- avoid leaving stale processes

Useful commands:

```bash
ps aux | grep ssh
pkill -f "ssh -f -N"
```

---

# SELinux Context Issues

## Problem

Services can fail even when Linux permissions look correct if SELinux contexts are wrong.

## Symptoms

- file exists
- Unix permissions look correct
- service still cannot access the file
- problem appears mysterious without SELinux context inspection

## Investigation

Check SELinux mode:

```bash
getenforce
```

Check file context:

```bash
ls -Z
```

Restore expected contexts:

```bash
sudo restorecon -Rv /path/to/directory
```

## Lesson Learned

On Fedora and other SELinux-enabled systems, permissions are not only Unix permissions.

Always consider:

- owner
- group
- mode
- SELinux context
- service domain

Disabling SELinux is not a fix. Understanding it is the fix.

---

# Nginx Sites Available vs Sites Enabled

## Problem

Nginx configuration appeared in both `sites-available` and `sites-enabled`, causing confusion about which file was active.

## Explanation

On Debian and Ubuntu systems:

```text
/etc/nginx/sites-available/
```

stores available virtual host configurations.

```text
/etc/nginx/sites-enabled/
```

contains symlinks to the configurations that are actually enabled.

Nginx usually loads files from `sites-enabled`.

## Useful Commands

```bash
ls -l /etc/nginx/sites-available/
ls -l /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## Lesson Learned

A backup file in `sites-available` is not necessarily active. The enabled symlink is what matters.

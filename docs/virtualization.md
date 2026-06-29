# Virtualization

## Overview

Virtualization is used to create isolated Linux environments for testing, troubleshooting, and system administration practice.

The homelab uses:

- KVM
- QEMU
- libvirt
- Virt-Manager
- virsh
- qcow2 disk images

## Hosts

Virtualization is used mainly on:

- Fedora Workstation
- Fedora Home Server

## Main Use Cases

- testing Linux distributions
- practicing server administration
- reproducing networking problems
- testing firewall behavior
- experimenting with SELinux
- validating service configurations
- creating isolated labs without affecting the main system

## KVM / QEMU / Libvirt

KVM provides kernel-level virtualization.

QEMU provides machine emulation.

Libvirt provides management tooling and APIs.

Virt-Manager provides a graphical management interface.

Together, they allow creating and managing virtual machines efficiently on Linux.

## Virt-Manager

Virt-Manager is used from the workstation to manage local and remote virtual machines.

Use cases:

- create VMs
- start/stop VMs
- inspect virtual hardware
- manage storage
- manage virtual networks
- connect to remote libvirt hosts

## Storage

Virtual machine disks use qcow2 images.

Typical storage locations:

```text
/var/lib/libvirt/images
```

A separate ISO storage pool may also be used for installation media.

Important points:

- disk ownership must be correct
- SELinux contexts must be correct
- libvirt storage pools must be active
- images must be readable by the qemu/libvirt process

Useful commands:

```bash
virsh pool-list --all
virsh vol-list default
ls -lh /var/lib/libvirt/images
sudo restorecon -Rv /var/lib/libvirt/images
```

## Networking

Libvirt commonly creates a NAT network using `virbr0`.

Typical components:

- virtual bridge
- DHCP
- dnsmasq
- NAT
- forwarding
- masquerading

Useful commands:

```bash
virsh net-list --all
virsh net-info default
ip addr show virbr0
sudo nft list ruleset
sudo firewall-cmd --list-all
```

## Common Problem: VM Gets IP but Has No Internet

A VM may receive an IP address from libvirt DHCP and still fail to access the internet.

Possible causes:

- forwarding disabled
- masquerade missing
- firewalld zone not forwarding traffic
- libvirt network inactive
- nftables rules missing
- wrong interface used for outbound traffic

Useful checks:

```bash
ip addr
ip route
ping 8.8.8.8
ping google.com
sudo sysctl net.ipv4.ip_forward
sudo nft list ruleset
```

## Remote Management

Remote libvirt can be managed through SSH.

Concept:

```text
Workstation
   |
   v
SSH
   |
   v
Remote libvirt host
```

This allows using the workstation as the control point while running VMs on the home server.

## Lessons Learned

Important lessons from virtualization work:

- DHCP success does not guarantee internet access.
- NAT depends on forwarding and masquerading.
- firewalld and libvirt must cooperate.
- SELinux context problems can break VM storage access.
- virt-manager and virsh are complementary tools.
- troubleshooting should start from the VM and move outward: guest, bridge, host firewall, NAT, upstream interface.

# vm-devops-lab-setup
Oracle VirtualBox DevOps lab covering Linux VMs, networking, NAT, Bridged, Host-only, SSH, port forwarding, snapshots, and multi-VM connectivity.

# VirtualBox DevOps Practice Lab

## Overview

This repository documents my hands-on practice setting up a DevOps lab using Oracle VirtualBox and Linux virtual machines.

It covers virtual machine creation, networking configuration, internet connectivity, SSH access, port forwarding, snapshots, and communication between multiple virtual machines.

The goal is to build a strong foundation in Linux administration and networking for a career in Cloud and DevOps engineering.

## Learning Objectives

- Create and configure Linux virtual machines.
- Understand virtual disks, ISO images, RAM, and virtual CPUs.
- Configure VirtualBox network adapters.
- Understand NAT, NAT Network, Bridged Adapter, Host-only, and Internal Network.
- Configure internet connectivity and troubleshoot IP address issues.
- Connect to Linux VMs using SSH and MobaXterm.
- Understand port forwarding and port binding.
- Connect multiple VMs for web, application, and database lab environments.
- Create and restore VM snapshots.
- Practise troubleshooting network connectivity.

## Technologies and Tools

- Oracle VirtualBox
- CentOS 7 for legacy lab practice
- Linux command line
- MobaXterm
- SSH
- TCP/IP networking
- DHCP and DNS
- Git and GitHub

## VirtualBox Networking

| Network Mode | Main Purpose |
|---|---|
| NAT | Provide outbound network connectivity to a VM |
| NAT Network | Connect multiple VMs and provide outbound connectivity |
| Bridged Adapter | Connect a VM to the physical LAN |
| Host-only Adapter | Private communication between the host and VMs |
| Internal Network | Isolated communication between VMs |
| Host-only + NAT | Provide private host access and outbound connectivity |

## Lab Architecture

```text
Windows Host
     |
 Oracle VirtualBox
     |
     +---- Web Server VM
     |
     +---- Application Server VM
     |
     +---- Database Server VM
```

The VMs can be connected using an appropriate shared virtual network based on the lab requirements.

## Useful Linux Commands

### Check IP addresses

```bash
ip addr
ip -4 addr
```

### Check routing

```bash
ip route
```

### Check network interfaces

```bash
nmcli device status
nmcli connection show
```

### Check SSH

```bash
sudo systemctl status sshd
```

### Check listening ports

```bash
sudo ss -lntp
```

## Hands-on Practice

1. Create a Linux VM using an ISO or an existing virtual disk.
2. Configure a NAT or Bridged Adapter.
3. Verify the VM's IPv4 address and default gateway.
4. Connect to the VM using MobaXterm and SSH.
5. Configure NAT port forwarding.
6. Create two VMs and connect them using a shared virtual network.
7. Test connectivity between the VMs.
8. Create a snapshot before changing system configuration.
9. Restore a snapshot and verify the VM state.
10. Troubleshoot network, routing, DNS, and SSH issues.

## Key Learnings

This lab helps develop practical skills in:

- Linux system administration
- Virtualization fundamentals
- IP addressing and routing
- Virtual networking
- SSH remote access
- Network troubleshooting
- Multi-VM lab design
- VM snapshots and recovery

## Future Improvements

- Build a three-tier web application lab.
- Practise Docker and Docker Compose.
- Configure private networks for application and database servers.
- Automate VM configuration using Ansible.
- Practise CI/CD workflows.
- Expand the lab for Kubernetes learning.

## Security Notes

- Use snapshots before major configuration changes.
- Keep separate backups of important virtual machines.
- Do not expose SSH or database services unnecessarily.
- Do not commit passwords, private keys, or other secrets to GitHub.
- Use supported Linux distributions for new projects.

## Project Status

**Status:** Ongoing hands-on learning project.

**Focus:** VirtualBox, Linux, networking, SSH, and DevOps lab infrastructure.


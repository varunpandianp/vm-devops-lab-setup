# DevOps Practice Lab Setup — VirtualBox, Linux VMs, Networking, SSH, and Snapshots

## 1. Project Overview

This document explains how to build a local DevOps practice environment using Oracle VirtualBox and Linux virtual machines.

The lab can be used to practise Linux administration, networking, SSH, Git, web servers, databases, Docker, and other DevOps technologies.

### Learning objectives

- Create and configure virtual machines.
- Understand virtual CPUs, RAM, virtual disks, and ISO images.
- Understand VirtualBox network adapters and network modes.
- Configure NAT, NAT Network, Bridged, Host-only, and Internal networking.
- Connect multiple VMs to each other.
- Configure internet connectivity.
- Access Linux VMs through MobaXterm using SSH.
- Understand port forwarding and application port binding.
- Take, restore, and manage VM snapshots.
- Troubleshoot IP address, routing, DNS, firewall, and SSH problems.

## 2. Virtualization Fundamentals

### What is virtualization?

Virtualization allows multiple virtual computers to run on one physical computer.

The physical computer is called the **host machine**. Each virtual computer is called a **virtual machine (VM)**.

Example:

```text
Physical Computer — Windows Host
          |
          v
    Oracle VirtualBox
          |
     +----+-----+
     |    |     |
     v    v     v
    VM1  VM2   VM3
   Linux Linux Linux
   Web   App   DB
```

### Important terminology

| Term | Meaning |
|---|---|
| Host machine | Your physical Windows computer |
| Guest OS | Operating system running inside a VM |
| Hypervisor | Software that runs and manages VMs |
| VirtualBox | A hosted, or Type 2, hypervisor |
| VM | Virtual computer with allocated resources |
| ISO | Disk image used to install an operating system |
| VDI | VirtualBox virtual hard disk format |
| VMDK | Virtual disk format also used by VMware |
| OVA | Packaged virtual appliance that can include a VM configuration and disks |
| Snapshot | Saved VM state used to return to an earlier point |
| Virtual NIC | Virtual network interface card inside a VM |
| DHCP | Protocol that automatically provides IP configuration |
| NAT | Network Address Translation |
| SSH | Secure Shell for remote terminal access |

### ISO vs VDI

**ISO:** Typically used to boot an installer and install an operating system.

**VDI:** A virtual hard disk that can contain an installed operating system or be empty.

If you download a VDI that already contains CentOS, you can attach it to a VM and boot it without installing CentOS from an ISO.

## 3. Creating Your First Virtual Machine

### Step 1: Prepare the VM

Open VirtualBox and select **New**.

Example settings for a lightweight Linux learning VM:

| Setting | Example |
|---|---|
| VM name | CentOS7-Lab |
| OS type | Linux |
| Version | Red Hat (64-bit), if applicable |
| RAM | 2 GB or more, depending on available memory |
| CPUs | 2, if available |
| Virtual disk | Existing VDI or a new virtual disk |

The correct resources depend on your physical computer and the operating system you intend to run.

### Step 2: Use an existing VDI

If you downloaded an existing CentOS VDI:

1. Create a new VM.
2. Select the option to use an existing virtual hard disk.
3. Browse to the downloaded `.vdi`.
4. Select the disk and finish the wizard.
5. Start the VM.

If you already created the VM, open **Settings → Storage → Add Hard Disk → Choose Existing Disk**.

Do not delete a disk file when you only intend to detach it from a VM.

### Step 3: Install from an ISO

If you have an empty virtual disk:

1. Create a new VM.
2. Select the appropriate Linux operating system.
3. Attach the installation ISO.
4. Allocate RAM, CPU, and disk space.
5. Boot the VM.
6. Follow the Linux installation wizard.

CentOS 7 is end-of-life. It can be useful for legacy training labs, but use a supported Linux distribution for new production-oriented practice where possible.

## 4. Understanding VirtualBox Networking

VirtualBox provides virtual network adapters that connect guest VMs to other VMs, your Windows host, and external networks.

### Network communication diagram

```text
Windows Host
     |
     v
VirtualBox Network Adapter
     |
     v
Guest Linux VM
     |
     +---- IP address
     +---- Subnet mask
     +---- Default gateway
     +---- DNS configuration
```

Each virtual network mode determines which systems can communicate and how traffic moves between them.

## 5. VirtualBox Network Modes

### 5.1 NAT — Network Address Translation

**Purpose:** Give a VM outbound network and internet access without making it directly accessible to other LAN computers.

Typical behavior:

- VM can usually access the internet.
- VM can usually initiate connections to external systems.
- Other VMs using ordinary, separate NAT instances cannot generally communicate directly.
- Windows cannot normally initiate a direct connection to the guest without port forwarding.
- Other computers on the LAN cannot normally initiate direct connections to the guest.

Example:

```text
Internet
   |
Router
   |
Windows Host
   |
VirtualBox NAT
   |
CentOS VM: 10.0.2.15
```

The address `10.0.2.15` is a common default for the first guest interface using VirtualBox NAT, but it is not guaranteed in every configuration.

#### Configure NAT

1. Shut down the VM.
2. Open **Settings → Network → Adapter 1**.
3. Enable Network Adapter.
4. Set **Attached to: NAT**.
5. Start the VM.

Check the guest network:

```bash
ip -4 addr
ip route
```

Test outbound connectivity:

```bash
ping -c 4 8.8.8.8
```

A failed ping does not necessarily mean internet access is broken because some networks block ICMP. You can also test DNS:

```bash
getent hosts example.com
```

### 5.2 NAT Network

**Purpose:** Allow multiple VMs on the same configured NAT Network to communicate with one another while also providing outbound connectivity.

Example:

```text
             Internet
                |
           VirtualBox NAT
              Network
             /       \
            /         \
       Web VM        DB VM
     10.0.2.x       10.0.2.y
```

VMs attached to the same NAT Network can generally communicate with each other, subject to firewall and service configuration.

#### Configure NAT Network

1. Open VirtualBox Manager.
2. Open the global **Tools → Network** or **Network Manager** section; the location varies by version.
3. Create a NAT Network.
4. Give it a name, such as `DevOps-NAT`.
5. Configure its network range, for example `10.10.10.0/24`, if supported by your version.
6. Enable DHCP if you want automatic IP assignment.
7. Open each VM's **Settings → Network**.
8. Set its adapter to **NAT Network** and select `DevOps-NAT`.
9. Start the VMs.

Example address plan:

| VM | Example IP |
|---|---|
| Web server | 10.10.10.10 |
| Application server | 10.10.10.20 |
| Database server | 10.10.10.30 |

These are example addresses, not guaranteed DHCP assignments. Use static addressing or DHCP reservations when you need stable addresses, and ensure they match your actual configured subnet.

### 5.3 Bridged Adapter

**Purpose:** Make the VM behave like another computer on your physical LAN.

Example:

```text
               Router
                 |
       +---------+---------+
       |                   |
 Windows Host           CentOS VM
 192.168.1.x            192.168.1.102
```

Your current CentOS VM uses this mode and received:

```text
192.168.1.102/24
```

Its default gateway is:

```text
192.168.1.1
```

The address was assigned through DHCP.

#### Configure Bridged Adapter

1. Shut down the VM.
2. Open **Settings → Network → Adapter 1**.
3. Select **Bridged Adapter**.
4. Select the actual active Wi-Fi or Ethernet adapter on Windows.
5. Enable **Cable Connected**.
6. Start the VM.
7. Run:

```bash
ip -4 addr show enp0s3
ip route
```

#### Advantages

- Easy access from MobaXterm using the guest's LAN IP.
- Other VMs on the LAN can communicate with it.
- The VM can communicate with physical systems, subject to network policies.

#### Disadvantages

- The VM is more directly exposed to your local network.
- Wi-Fi bridging, VPNs, corporate networks, and router policies can interfere with connectivity.
- The guest IP may change when its DHCP lease changes.

Do not expose SSH or other services to untrusted networks without appropriate security controls.

### 5.4 Host-only Adapter

**Purpose:** Create a private network between your Windows host and VMs attached to the same host-only network.

Example:

```text
       Windows Host
             |
       Host-only Network
          /       \
         /         \
    CentOS VM    Ubuntu VM
    192.168.56.x 192.168.56.y
```

Host-only networking normally does not provide internet access by itself.

#### Configure Host-only

1. Open VirtualBox's global network manager.
2. Create a Host-only network.
3. Note its IPv4 address and subnet.
4. Enable the host's virtual adapter and DHCP server if required.
5. Open each VM's **Settings → Network**.
6. Choose **Host-only Adapter**.
7. Select the same host-only network for the VMs.
8. Start the VMs and inspect their addresses.

Example subnet:

```text
192.168.56.0/24
```

The exact subnet depends on your host-only network configuration.

#### Use cases

- Connect to Linux VMs from MobaXterm on Windows.
- Practise SSH without exposing the VM directly to the physical LAN.
- Connect multiple private lab VMs.

### 5.5 Internal Network

**Purpose:** Create an isolated network for VMs that share the same internal network name.

Example:

```text
      Internal Network: devops-lab
             /       \
            /         \
        Web VM       DB VM
```

The VMs can communicate with each other, but the Windows host cannot normally access the network directly. Internet access is not provided by default.

#### Configure Internal Network

1. Shut down the VMs.
2. Open each VM's **Settings → Network**.
3. Choose **Internal Network**.
4. Enter the same network name, such as `devops-lab`.
5. Start the VMs.
6. Configure IP addresses using DHCP from a VM acting as a DHCP server or manually.

Example static addressing:

| VM | IP address | Subnet mask |
|---|---|---|
| Web VM | 10.20.0.10 | 255.255.255.0 |
| DB VM | 10.20.0.20 | 255.255.255.0 |

For this isolated example, both VMs can communicate directly over the subnet. They will not have internet access unless you deliberately add and configure a routing solution.

### 5.6 Host-only + NAT

**Purpose:** Give a VM internet access and a separate private connection to Windows or other lab VMs.

Configure two adapters:

- Adapter 1: NAT
- Adapter 2: Host-only Adapter

Example:

```text
                    Internet
                       |
                      NAT
                       |
                 +-----+-----+
                 | CentOS VM |
                 +-----+-----+
                       |
                  Host-only
                       |
                  Windows Host
```

The VM has two virtual NICs, usually with different interface names.

Example:

```text
enp0s3 = NAT interface
enp0s8 = Host-only interface
```

Actual names may vary.

Run:

```bash
ip -4 addr
ip route
```

Use the NAT interface for outbound internet connectivity and the host-only IP for SSH from Windows.

Avoid configuring two competing default gateways unless you understand route metrics and routing behavior.

## 6. Quick Comparison of Network Modes

| Mode | Guest internet | VM-to-VM | Host to VM | Other LAN systems to VM |
|---|---|---|---|---|
| NAT | Usually yes | Not normally across separate NAT instances | Requires forwarding for inbound access | Requires forwarding |
| NAT Network | Usually yes | Yes, on same network | Not directly by default | Not directly by default |
| Host-only | No, by default | Yes, on same network | Yes | No |
| Host-only + NAT | Yes, through NAT | Yes, on shared host-only network | Yes | No |
| Bridged | Usually yes | Yes, if network permits | Yes | Yes, if network permits |
| Internal Network | No, by default | Yes, on same named network | No direct access | No direct access |

Communication also depends on IP configuration, routing, firewalls, and the services running on the guest.

## 7. What Is an IP Address?

An IP address identifies a network interface on an IP network.

Example:

```text
192.168.1.102/24
```

- `192.168.1.102` is the interface's IPv4 address.
- `/24` means 24 bits are used for the network prefix.
- The corresponding subnet mask is `255.255.255.0`.

For the subnet `192.168.1.0/24`:

- Network address: `192.168.1.0`
- Typical host range: `192.168.1.1` to `192.168.1.254`
- Broadcast address: `192.168.1.255`

The actual usable host range depends on the subnet and any addresses reserved for gateways or other infrastructure.

### Important Linux network commands

```bash
ip addr
ip -4 addr
ip route
ip link
nmcli device status
nmcli connection show
```

`ip addr` displays addresses and interface details.

`ip route` displays routing information.

`nmcli` manages NetworkManager connections on systems that use NetworkManager.

## 8. Understanding DHCP

DHCP automatically provides network configuration to a client.

It can supply:

- IPv4 address
- Subnet mask or prefix length
- Default gateway
- DNS servers
- Lease duration

Basic DHCP process:

```text
Client VM             DHCP Server
   |                       |
   |------ Discover ------>|
   |<------- Offer --------|
   |------ Request ------->|
   |<------- ACK ----------|
```

This is often summarized as DORA: Discover, Offer, Request, Acknowledgment.

In your lab, `enp0s3` initially had no active connection profile, so it did not receive an IPv4 address. After you created and activated a NetworkManager profile with automatic IPv4 configuration, it received `192.168.1.102`.

Example:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname enp0s3 \
  con-name wired-enp0s3 \
  ipv4.method auto

sudo nmcli connection up wired-enp0s3
```

Check the result:

```bash
ip -4 addr show enp0s3
ip route
```

Do not recreate a connection profile unnecessarily if the existing profile already works.

## 9. Connecting Multiple VMs

For a simple multi-tier DevOps lab, use three VMs:

- Web server
- Application server
- Database server

### Recommended practice topology

For isolated communication and internet access, use a shared NAT Network for the VMs. If you also need to access each VM privately from Windows, add a Host-only adapter.

```text
Windows Host
     |
     +------ Host-only Network
     |          |       |       |
     |        Web VM   App VM   DB VM
     |          |       |       |
     +----------+-------+-------+
                |
          Shared NAT Network
                |
             Internet
```

Each VM must have the appropriate adapter configured for the network shown. The diagram represents two network paths: a private host-only path and a shared NAT Network path.

### Example service flow

```text
Browser
   |
   v
Web Server
   |
   v
Application Server
   |
   v
Database Server
```

Example service ports:

| Service | Common port |
|---|---:|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| MySQL/MariaDB | 3306 |
| PostgreSQL | 5432 |
| Redis | 6379 |

Only expose the ports your lab needs. Database services should generally be reachable only from trusted application hosts or the private lab network.

### Configure multiple VMs on the same network

1. Create the VMs.
2. Configure each VM to use the same NAT Network, Host-only network, or Internal Network, as required.
3. Start the VMs.
4. Check their IP addresses.
5. Test connectivity between them.
6. Verify firewall rules and service availability.

Example:

```bash
ip -4 addr
ip route
ping -c 4 10.10.10.20
```

Replace the example address with the real IP of the target VM. Ping can fail even when a host is reachable if ICMP is blocked.

## 10. Port Forwarding vs Port Binding

These are related concepts, but they are not the same.

### Port forwarding

Port forwarding directs incoming traffic arriving at a host address and port to a guest VM address and port.

Example:

```text
MobaXterm
    |
    v
Windows Host: 127.0.0.1:2222
    |
    | VirtualBox NAT Port Forwarding
    v
CentOS VM: Guest Port 22 (SSH)
```

Here:

- Host IP: `127.0.0.1`
- Host port: `2222`
- Guest port: `22`

The host uses port `2222`, while SSH continues to listen on port `22` inside CentOS.

### Configure VirtualBox NAT port forwarding

1. Shut down the VM if required by your VirtualBox version.
2. Open **Settings → Network → Adapter 1**.
3. Select NAT.
4. Open **Advanced → Port Forwarding**.
5. Add a rule:

| Field | Value |
|---|---|
| Name | SSH |
| Protocol | TCP |
| Host IP | 127.0.0.1 |
| Host Port | 2222 |
| Guest IP | Leave blank unless needed |
| Guest Port | 22 |

6. Start the VM.
7. Ensure the SSH service is running.

Inside CentOS:

```bash
sudo systemctl enable --now sshd
sudo systemctl status sshd
```

From MobaXterm:

- Session type: SSH
- Remote host: `127.0.0.1`
- Port: `2222`
- Username: your CentOS account

This works when the port-forwarding rule and guest SSH service are correctly configured.

### What is port binding?

Port binding usually refers to an application binding a socket to an IP address and port on which it listens.

For example:

```bash
sudo ss -lntp
```

This lists listening TCP sockets.

Example application binding:

```text
0.0.0.0:80
```

This usually means the application listens on all IPv4 interfaces.

```text
127.0.0.1:80
```

This means the application listens only on the loopback interface and is not directly reachable through the guest's external IP.

**Remember:**

- Port binding: an application listens on an address and port.
- Port forwarding: network traffic is redirected from one address and port to another.

### Docker port publishing

Docker has its own port-publishing mechanism, often expressed as:

```bash
docker run -d -p 8080:80 nginx
```

This publishes host port `8080` to container port `80`.

Docker publishing is different from VirtualBox NAT port forwarding, although the two can be used together in nested network setups.

## 11. Internet Connectivity Troubleshooting

When a VM cannot access the internet, check the problem in layers.

### Layer 1: Interface

```bash
ip link
nmcli device status
```

Confirm the expected network interface exists and is connected.

### Layer 2: IPv4 address

```bash
ip -4 addr
```

Confirm the interface has a valid address for its configured network mode.

### Layer 3: Routing

```bash
ip route
```

Check whether a suitable default route exists.

Example:

```text
default via 192.168.1.1 dev enp0s3
```

The gateway must be valid for that interface and subnet.

### Layer 4: Gateway connectivity

```bash
ping -c 4 192.168.1.1
```

Use the actual gateway shown by `ip route`.

### Layer 5: External IP connectivity

```bash
ping -c 4 8.8.8.8
```

If this fails, check the network mode, gateway, upstream connectivity, and firewall rules. ICMP may be blocked.

### Layer 6: DNS resolution

```bash
getent hosts example.com
cat /etc/resolv.conf
```

If external IP connectivity works but domain names do not resolve, investigate DNS configuration.

### Common problem: no IPv4 address

Symptoms:

```text
enp0s3: disconnected
```

or no `inet` line under the interface.

Possible causes:

- VirtualBox adapter is disabled.
- Cable Connected is unchecked.
- Wrong physical adapter selected in Bridged mode.
- DHCP is unavailable.
- NetworkManager is not managing the interface.
- No active connection profile exists.

Check:

```bash
sudo systemctl status NetworkManager
nmcli device status
nmcli connection show
ip -4 addr
```

If appropriate, activate an existing connection profile:

```bash
sudo nmcli connection up "CONNECTION_NAME"
```

Replace `CONNECTION_NAME` with the actual profile name.

### Common problem: VM has an IP but cannot reach the internet

Check:

- Default route
- DNS
- Host internet connectivity
- VirtualBox network mode
- Firewall and network policies
- Whether the network requires a proxy

An IP address alone does not prove internet access is working.

## 12. SSH and MobaXterm

SSH allows secure remote command-line access to a Linux VM.

### Check SSH service

On CentOS:

```bash
sudo systemctl status sshd
```

Enable and start it:

```bash
sudo systemctl enable --now sshd
```

If `sshd` is not found, the OpenSSH server package may need to be installed.

### Connect using Bridged networking

If the VM IP is `192.168.1.102`:

- Remote host: `192.168.1.102`
- Port: `22`
- Username: your Linux account, such as `osboxes`

The host and guest must have a working network path, and the guest firewall must allow SSH.

### Connect using NAT port forwarding

- Remote host: `127.0.0.1`
- Port: `2222`
- Username: your Linux account

This requires the VirtualBox forwarding rule from host port `2222` to guest port `22`.

### Troubleshoot SSH

Check service:

```bash
sudo systemctl status sshd
```

Check listening port:

```bash
sudo ss -lntp | grep ':22'
```

Check firewall services, if firewalld is installed and active:

```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-services
```

If required, allow SSH on the trusted lab network:

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

Do not disable the firewall indiscriminately. Configure only the access you need.

## 13. VM Snapshots

### What is a snapshot?

A snapshot records a VM's state at a point in time, allowing you to return to that state later.

Snapshots are useful before:

- Installing software
- Changing network settings
- Editing system configuration
- Installing Docker or Kubernetes prerequisites
- Performing potentially risky experiments

### Create a snapshot

1. Shut down or pause the VM if appropriate for the change.
2. In VirtualBox Manager, select the VM.
3. Open the **Snapshots** view.
4. Click **Take** or **Take Snapshot**.
5. Enter a descriptive name.

Example snapshot names:

```text
01-clean-centos
02-network-configured
03-ssh-working
04-docker-installed
```

### Restore a snapshot

1. Select the VM.
2. Open Snapshots.
3. Select the snapshot.
4. Choose Restore.
5. Confirm the operation.

Restoring a snapshot can discard changes made after the snapshot, depending on the restore choices.

### Snapshot vs backup

A snapshot is not a substitute for a separate backup. Snapshots depend on the VM's disk and snapshot chain and may consume significant storage.

For important lab configurations, keep a separate backup of the virtual disk or export the VM when appropriate.

### Practical snapshot strategy

```text
Create VM
   |
   v
Install Linux
   |
   v
Snapshot: clean-install
   |
   v
Configure networking and SSH
   |
   v
Snapshot: network-working
   |
   v
Install DevOps tools
   |
   v
Snapshot: tools-installed
```

Use meaningful names and delete obsolete snapshots carefully.

## 14. Recommended DevOps Lab Architecture

Start small and increase complexity gradually.

### Stage 1: Single VM

Create one Linux VM and practise:

- Linux commands
- Users and permissions
- Package management
- Services and systemd
- IP addressing and routing
- SSH
- Git

Recommended network: NAT or Bridged, depending on your access requirements.

### Stage 2: Two VMs

Create:

- Web VM
- Database VM

Connect both to the same NAT Network or Internal Network.

Practise:

- Assigning IP addresses
- Testing connectivity
- Configuring SSH
- Installing Apache or Nginx
- Installing MariaDB
- Configuring firewall rules
- Restricting database access to the web/application VM

### Stage 3: Three VMs

Create:

- Web server
- Application server
- Database server

Practise:

- Multi-tier architecture
- Service-to-service connectivity
- Name resolution
- Firewall rules
- Logs and troubleshooting
- Application deployment

### Stage 4: Container lab

Install Docker on a suitable Linux VM.

Practise:

- Docker images and containers
- Port publishing
- Bridge networks
- Container-to-container communication
- Docker Compose

Take a VM snapshot before major changes.

## 15. Hands-on Exercises

### Exercise 1: Identify your IP configuration

Run:

```bash
ip -4 addr
ip route
nmcli device status
```

Record:

- Interface name
- IP address and prefix
- Default gateway
- Network mode
- Whether DHCP is being used

### Exercise 2: Compare NAT and Bridged

1. Take a snapshot.
2. Switch the VM to NAT.
3. Boot and record the IP address.
4. Test internet connectivity.
5. Switch to Bridged Adapter.
6. Boot and record the new IP address.
7. Compare routing and SSH access.

Do not assume the VM's IP will remain the same after changing network modes.

### Exercise 3: Connect two VMs

1. Create two Linux VMs.
2. Attach both to the same NAT Network.
3. Boot them.
4. Find their IP addresses.
5. Test connectivity.
6. Start SSH on one VM.
7. Connect from the other VM.

### Exercise 4: Configure SSH port forwarding

1. Set Adapter 1 to NAT.
2. Add a forwarding rule from host port `2222` to guest port `22`.
3. Start SSH in the guest.
4. Connect to `127.0.0.1:2222` from MobaXterm.
5. Confirm that the connection works.

### Exercise 5: Practise snapshots

1. Create a snapshot called `before-change`.
2. Make a safe configuration change.
3. Verify the change.
4. Restore the snapshot.
5. Confirm that the VM returned to its earlier state.

## 16. Common Questions and Answers

### Why does my VM not have an IP address?

The virtual interface may be disconnected, DHCP may be unavailable, or NetworkManager may lack an active connection profile.

Check:

```bash
nmcli device status
nmcli connection show
ip -4 addr
```

### Why can the VM access the internet but Windows cannot connect to it?

The VM may be using NAT. NAT permits outbound traffic by default but normally requires port forwarding for direct inbound access.

### Why can two VMs not communicate?

They may be on different virtual networks or subnets. A firewall, missing route, or inactive service may also block communication.

### Why is my Bridged Adapter not working over Wi-Fi?

The selected Windows adapter may be incorrect, or the Wi-Fi driver, VPN, access point, or network policy may interfere with bridging. Try the correct active adapter or use NAT plus Host-only networking.

### What is the difference between Host-only and Internal Network?

Host-only generally connects VMs to the Windows host through a host virtual adapter. Internal Network connects VMs to each other without directly exposing that network to the host.

### Can NAT provide internet and private VM-to-VM communication?

Ordinary NAT instances are generally isolated from one another. Use a shared NAT Network or another shared virtual network when you need multiple VMs to communicate.

### What is the difference between a snapshot and a clone?

A snapshot captures a point in the history of one VM. A clone creates another VM based on an existing VM, with its own configuration and virtual disk relationship depending on the clone type.

### What is the difference between a port and an IP address?

An IP address identifies a network interface. A port identifies a service endpoint on a host, using a transport protocol such as TCP or UDP.

For example, `192.168.1.102:22` refers to TCP port 22 on the host with IP `192.168.1.102`.

### Does a successful ping prove that SSH works?

No. Ping tests ICMP reachability. SSH requires the SSH server to be running, listening on the appropriate port, reachable through the network, and allowed by firewall rules.

## 17. Troubleshooting Checklist

When a VM cannot connect to another system, investigate in this order:

1. Is the VM running?
2. Is the correct VirtualBox adapter enabled?
3. Is the cable connected?
4. Is the expected interface up?
5. Does it have a valid IP address?
6. Is the subnet correct?
7. Is the default gateway correct when required?
8. Are the two VMs attached to a network that permits communication?
9. Can the guest reach the destination IP?
10. Is the required service running?
11. Is the service listening on the expected port?
12. Do firewall rules permit the connection?
13. Is DNS resolving the correct address?
14. Are VirtualBox port-forwarding rules configured correctly, if required?

Useful commands:

```bash
ip addr
ip route
nmcli device status
nmcli connection show
getent hosts example.com
sudo systemctl status sshd
sudo ss -lntp
```

## 18. Security Best Practices

- Use snapshots before risky experiments.
- Keep separate backups of important VMs.
- Do not expose SSH to untrusted networks unnecessarily.
- Use strong passwords or SSH keys.
- Do not publish secrets in GitHub repositories.
- Restrict database access to trusted hosts.
- Open only required firewall ports.
- Avoid bridged networking on untrusted public networks.
- Use isolated networks for experiments that could affect other systems.
- Prefer supported operating systems for current projects.

## 19. Learning Log Template

Record each lab session using this format:

### Date

YYYY-MM-DD

### Objective

What did I want to practise?

### Environment

- Hypervisor: Oracle VirtualBox
- Guest OS:
- Network mode:
- VM names:

### Commands used

```bash
# Add the commands used in this session.
```

### What I learned

-
-
-

### Problems encountered

Describe the symptom and error message.

### Root cause

What caused the issue?

### Solution

What steps resolved it?

### Next practice task

What will I practise next?

## 20. Summary

VirtualBox networking is a foundation for understanding Linux networking and cloud infrastructure.

Remember these core distinctions:

- **NAT:** Simple outbound connectivity for a VM.
- **NAT Network:** Shared network for multiple VMs with outbound connectivity.
- **Bridged Adapter:** VM joins the physical LAN.
- **Host-only:** Private connectivity between the host and VMs.
- **Internal Network:** Isolated communication among VMs on the same named network.
- **Host-only + NAT:** Private host access plus outbound connectivity.
- **Port forwarding:** Redirect incoming traffic to a guest service.
- **Port binding:** An application listens on a particular address and port.
- **Snapshot:** A recovery point in a VM's history.

Build the lab one step at a time, verify every network change, and document the actual IP addresses and commands you use. These habits are directly useful when troubleshooting servers in cloud and DevOps environments.
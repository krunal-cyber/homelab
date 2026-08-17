# Configuration of Virtual Network for Multi-VM Communication

## Aim

To configure networking between virtual machines using VMware NAT and Local Network to enable internal communication and Internet access.

## Objectives

* Configure NAT networking for Internet connectivity.
* Configure Local Network for communication between virtual machines.
* Verify network interfaces.
* Test communication among all virtual machines.

## Network Topology

!\[Figure 1.5 – Network topology](./screenshots/network-topology.jpeg)
*Figure 1.5 – Network topology*

## Network Configuration

### Ubuntu Server

* **Adapter 1** — Type: NAT · Purpose: Internet connectivity · IP Assignment: DHCP
* **Adapter 2** — Type: Local Network · Purpose: Internal communication · IP Assignment: Static (configured later for lab experiments)

### Windows 11

* **Adapter** — Type: Local Network · Purpose: Client system

### Parrot Security OS

* **Adapter** — Type: Local Network · Purpose: Security testing workstation

### Metasploitable 2

* **Adapter** — Type: Local Network · Purpose: Vulnerable target machine

## Procedure

1. Configured Ubuntu Server with two network adapters.
2. Connected the first adapter to the VMware NAT Network.
3. Connected the second adapter to the Local Network.
4. Connected Windows 11, Parrot OS, and Metasploitable 2 to the same Local Network.
5. Verified the network interfaces using `ip a` / `ip route` (Ubuntu, Parrot) and `ipconfig` (Windows).
6. Confirmed that Ubuntu had Internet access through the NAT adapter.
7. Verified that all client virtual machines were connected to the Local Network.

## Commands Used

**Ubuntu**

```
ip a
ip route
ping 8.8.8.8
```

**Windows**

```
ipconfig
ping <Ubuntu\_IP>
```

**Parrot**

```
ip a
ping <Ubuntu\_IP>
```

## Observations

* Ubuntu Server successfully obtained Internet connectivity through the NAT adapter.
* All virtual machines were connected to the same Local Network.
* Network interfaces were detected correctly on every VM.
* The lab environment was confirmed ready for static IP addressing, DHCP configuration, SSH communication, HTTP service testing, and further cybersecurity experiments.

## Result

The virtual network was successfully configured using VMware NAT and Local Network. Ubuntu Server functioned as the central machine with Internet connectivity, while Windows 11, Parrot OS, and Metasploitable 2 were connected to the internal laboratory network for future networking and cybersecurity practicals.

### Evidence

!\[Figure 1.1 – VMware network adapter settings](./screenshots/network-adapters.jpeg)
*Figure 1.1 – VMware network adapter settings*

!\[Figure 1.2 – Ubuntu ip a output](./screenshots/ubuntu-ip-a.png)
*Figure 1.2 – Output of `ip a` on Ubuntu*

!\[Figure 1.3 – Ubuntu ip route output](./screenshots/ubuntu-ip-route.png)
*Figure 1.3 – Output of `ip route` on Ubuntu*

!\[Figure 1.4 – Windows ipconfig output](./screenshots/windows-ipconfig.png)
*Figure 1.4 – Output of `ipconfig /all` on Windows*

!\[Figure 1.5 – Parrot ip a output](./screenshots/parrot-ip-a.png)
*Figure 1.5 – Output of `ip a` on Parrot*

!\[Figure 1.6 – Ubuntu pinging the internet](./screenshots/ubuntu-ping-internet.png)
*Figure 1.6 – Ubuntu successfully pinging the internet via NAT*

!\[Figure 1.7 – Parrot pinging other VMs](./screenshots/parrot-ping-vms.png)
*Figure 1.7 – Parrot successfully pinging other VMs on the Local Network*


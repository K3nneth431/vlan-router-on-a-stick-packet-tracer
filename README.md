# Inter-VLAN Routing Lab (Router-on-a-Stick) in Cisco Packet Tracer

A network lab built in Cisco Packet Tracer: four switches and 16 PCs split into four VLANs with different subnet sizes (VLSM). A single router connects the VLANs using the **router-on-a-stick** technique, and the network devices have protected management access.

> This is a learning project I built while studying networking (Systems and Networks course). The open file is `VLAN_router_on_stick.pkt`.

## What this lab shows

- Segmenting a network into **4 VLANs** (production, technical, administration, IT) spread across **4 switches**
- **VLSM** subnetting with `/24`, `/27`, `/28` and `/29` networks
- **Trunk links** between the switches and between the router and a switch
- **Router-on-a-stick**: one physical router link with one **subinterface per VLAN**, each acting as the default gateway of its VLAN
- Basic **device access hardening** (console password, VTY password, encrypted enable secret)

## Topology

![Network topology](topology.png)

- `Router0` is connected to `Switch0` through a single trunk link.
- The four switches are chained with trunk links: `Switch0 - Switch1 - Switch2 - Switch3`.
- Each switch has 4 PCs, one in each VLAN (10, 20, 30 and 270), so every VLAN spans all four switches.

## Addressing plan

Each VLAN is a department of a fictional company. The subnets are sized (VLSM) for the number of hosts each department needs.

| VLAN ID | Name | Hosts needed | Network | Subnet mask | Usable range | Gateway (subinterface) |
|---------|------|--------------|---------|-------------|--------------|------------------------|
| 10 | Produzione (production) | 200 | 192.168.0.0/24 | 255.255.255.0 | 192.168.0.1 - 192.168.0.254 | 192.168.0.254 |
| 20 | Tecnico (technical) | 30 | 192.168.1.0/27 | 255.255.255.224 | 192.168.1.1 - 192.168.1.30 | 192.168.1.30 |
| 30 | Amministrativo (administration) | 8 | 192.168.1.32/28 | 255.255.255.240 | 192.168.1.33 - 192.168.1.46 | 192.168.1.46 |
| 270 | IT | 3 | 192.168.1.48/29 | 255.255.255.248 | 192.168.1.49 - 192.168.1.54 | 192.168.1.54 |

### PC addresses

The topology simulates a few PCs per VLAN to represent each department.

| Switch | VLAN 10 | VLAN 20 | VLAN 30 | VLAN 270 |
|--------|---------|---------|---------|----------|
| Switch0 | 192.168.0.1/24 | 192.168.1.1/27 | 192.168.1.33/28 | 192.168.1.49/29 |
| Switch1 | 192.168.0.2/24 | 192.168.1.2/27 | 192.168.1.34/28 | 192.168.1.50/29 |
| Switch2 | 192.168.0.3/24 | 192.168.1.3/27 | 192.168.1.35/28 | 192.168.1.51/29 |
| Switch3 | 192.168.0.4/24 | 192.168.1.4/27 | 192.168.1.36/28 | 192.168.1.52/29 |

## How router-on-a-stick works here

The router has only one physical link to the switches. That link is a trunk that carries the traffic of all VLANs. On the router, the physical interface has no IP address and is simply turned on. For each VLAN there is a **subinterface** that:

1. tags and untags frames for that VLAN (`encapsulation dot1Q <VLAN ID>`), and
2. has an IP address in the VLAN's network, used as the **default gateway** of that VLAN's PCs.

When a PC wants to reach a PC in another VLAN, it sends the traffic to its gateway. The router receives it on one subinterface and sends it back out through the subinterface of the destination VLAN.

### Example subinterface configuration

```
interface gig9/0
 no shutdown
 exit

interface gig9/0.10
 encapsulation dot1Q 10
 ip address 192.168.0.254 255.255.255.0
 exit

interface gig9/0.20
 encapsulation dot1Q 20
 ip address 192.168.1.30 255.255.255.224
 exit

interface gig9/0.30
 encapsulation dot1Q 30
 ip address 192.168.1.46 255.255.255.240
 exit

interface gig9/0.270
 encapsulation dot1Q 270
 ip address 192.168.1.54 255.255.255.248
 exit
```

## Device access hardening

Management access to the devices is protected with a console password, an encrypted `enable secret` and a VTY password:

```
enable
configure terminal

service password-encryption

line console 0
 password <password>
 login
 exit

enable secret <password>

line vty 0 15
 password <password>
 login
 exit
```

> The passwords used in the lab are for learning purposes only and are not published here. 

## How to open the lab

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco Networking Academy account).
2. Download `VLAN_router_on_stick.pkt` from this repository.
3. Open it in Packet Tracer.

## Testing



I tested connectivity with `ping` from the command prompt of the PCs in Packet Tracer.

| Test | From | To | Result |
|------|------|----|--------|
| Same VLAN, different switches | PC in VLAN 10 on Switch0 (192.168.0.1) | PC in VLAN 10 on Switch3 (192.168.0.4) | Success |
| Different VLANs (through the router) | PC in VLAN 10 (192.168.0.1) | PC in VLAN 270 (192.168.1.49) | Success | -> in the png file

![Ping test](ping-test.png)

## What I learned

- How VLANs separate traffic on the same physical switches, and how trunks carry several VLANs on one link.
- How to plan addresses with VLSM so each subnet fits its number of hosts.
- How a single router link can provide inter-VLAN routing using subinterfaces.
- Why management access to network devices must be protected.

## Steps i might add in the future

- DHCP pools on the router, one for each VLAN
- SSH instead of Telnet for remote management
- Port security on the switches

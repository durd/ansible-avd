# FABRIC

## Table of Contents

- [Fabric Switches and Management IP](#fabric-switches-and-management-ip)
  - [Fabric Switches with inband Management IP](#fabric-switches-with-inband-management-ip)
- [Fabric Topology](#fabric-topology)
- [Fabric IP Allocation](#fabric-ip-allocation)
  - [Fabric Point-To-Point Links](#fabric-point-to-point-links)
  - [Point-To-Point Links Node Allocation](#point-to-point-links-node-allocation)
  - [Loopback Interfaces (BGP EVPN Peering)](#loopback-interfaces-bgp-evpn-peering)
  - [Loopback0 Interfaces Node Allocation](#loopback0-interfaces-node-allocation)
  - [VTEP Loopback VXLAN Tunnel Source Interfaces (VTEPs Only)](#vtep-loopback-vxlan-tunnel-source-interfaces-vteps-only)
  - [VTEP Loopback Node allocation](#vtep-loopback-node-allocation)

## Fabric Switches and Management IP

| POD | Type | Node | Management IP | Platform | Provisioned in CloudVision | Serial Number |
| --- | ---- | ---- | ------------- | -------- | -------------------------- | ------------- |
| FABRIC | l3leaf | dc1-leaf1a | 172.16.1.101/24 | 7280R3 | Provisioned | - |
| FABRIC | l3leaf | dc1-leaf1b | 172.16.1.102/24 | 7280R3 | Provisioned | - |
| FABRIC | l2leaf | dc1-leaf1c | 172.16.1.151/24 | vEOS-lab | Provisioned | - |
| FABRIC | l3leaf | dc2-leaf1a | 172.16.2.101/24 | 7280R3 | Provisioned | - |
| FABRIC | l3leaf | dc2-leaf1b | 172.16.2.102/24 | 7280R3 | Provisioned | - |
| FABRIC | l2leaf | dc2-leaf1c | 172.16.2.151/24 | vEOS-lab | Provisioned | - |

> Provision status is based on Ansible inventory declaration and do not represent real status from CloudVision.

### Fabric Switches with inband Management IP

| POD | Type | Node | Management IP | Inband Interface |
| --- | ---- | ---- | ------------- | ---------------- |

## Fabric Topology

| Type | Node | Node Interface | Peer Type | Peer Node | Peer Interface |
| ---- | ---- | -------------- | --------- | ----------| -------------- |
| l3leaf | dc1-leaf1a | Ethernet43 | l2leaf | dc1-leaf1c | Ethernet47 |
| l3leaf | dc1-leaf1a | Ethernet44 | l3leaf | dc2-leaf1a | Ethernet44 |
| l3leaf | dc1-leaf1a | Ethernet45 | l3leaf | dc2-leaf1b | Ethernet45 |
| l3leaf | dc1-leaf1a | Ethernet46 | l3leaf | dc1-leaf1b | Ethernet46 |
| l3leaf | dc1-leaf1a | Ethernet47 | mlag_peer | dc1-leaf1b | Ethernet47 |
| l3leaf | dc1-leaf1a | Ethernet48 | mlag_peer | dc1-leaf1b | Ethernet48 |
| l3leaf | dc1-leaf1b | Ethernet43 | l2leaf | dc1-leaf1c | Ethernet48 |
| l3leaf | dc1-leaf1b | Ethernet44 | l3leaf | dc2-leaf1b | Ethernet44 |
| l3leaf | dc1-leaf1b | Ethernet45 | l3leaf | dc2-leaf1a | Ethernet45 |
| l3leaf | dc2-leaf1a | Ethernet43 | l2leaf | dc2-leaf1c | Ethernet47 |
| l3leaf | dc2-leaf1a | Ethernet46 | l3leaf | dc2-leaf1b | Ethernet46 |
| l3leaf | dc2-leaf1a | Ethernet47 | mlag_peer | dc2-leaf1b | Ethernet47 |
| l3leaf | dc2-leaf1a | Ethernet48 | mlag_peer | dc2-leaf1b | Ethernet48 |
| l3leaf | dc2-leaf1b | Ethernet43 | l2leaf | dc2-leaf1c | Ethernet48 |

## Fabric IP Allocation

### Fabric Point-To-Point Links

| Uplink IPv4 Pool | Available Addresses | Assigned addresses | Assigned Address % |
| ---------------- | ------------------- | ------------------ | ------------------ |
| 10.255.12.0/24 | 256 | 4 | 1.57 % |
| 10.255.22.0/24 | 256 | 8 | 3.13 % |

### Point-To-Point Links Node Allocation

| Node | Node Interface | Node IP Address | Peer Node | Peer Interface | Peer IP Address |
| ---- | -------------- | --------------- | --------- | -------------- | --------------- |
| dc1-leaf1a | Ethernet44 | 10.255.22.40/31 | dc2-leaf1a | Ethernet44 | 10.255.22.41/31 |
| dc1-leaf1a | Ethernet45 | 10.255.22.128/31 | dc2-leaf1b | Ethernet45 | 10.255.22.129/31 |
| dc1-leaf1a | Ethernet46 | 10.255.12.44/31 | dc1-leaf1b | Ethernet46 | 10.255.12.45/31 |
| dc1-leaf1b | Ethernet44 | 10.255.22.130/31 | dc2-leaf1b | Ethernet44 | 10.255.22.131/31 |
| dc1-leaf1b | Ethernet45 | 10.255.12.47/31 | dc2-leaf1a | Ethernet45 | 10.255.12.46/31 |
| dc2-leaf1a | Ethernet46 | 10.255.22.126/31 | dc2-leaf1b | Ethernet46 | 10.255.22.127/31 |

### Loopback Interfaces (BGP EVPN Peering)

| Loopback Pool | Available Addresses | Assigned addresses | Assigned Address % |
| ------------- | ------------------- | ------------------ | ------------------ |
| 10.255.1.0/27 | 32 | 2 | 6.25 % |
| 10.255.2.0/27 | 32 | 2 | 6.25 % |

### Loopback0 Interfaces Node Allocation

| POD | Node | Loopback0 |
| --- | ---- | --------- |
| FABRIC | dc1-leaf1a | 10.255.1.13/32 |
| FABRIC | dc1-leaf1b | 10.255.1.14/32 |
| FABRIC | dc2-leaf1a | 10.255.2.23/32 |
| FABRIC | dc2-leaf1b | 10.255.2.24/32 |

### VTEP Loopback VXLAN Tunnel Source Interfaces (VTEPs Only)

| VTEP Loopback Pool | Available Addresses | Assigned addresses | Assigned Address % |
| --------------------- | ------------------- | ------------------ | ------------------ |
| 10.255.11.0/24 | 256 | 2 | 0.79 % |
| 10.255.21.0/24 | 256 | 2 | 0.79 % |

### VTEP Loopback Node allocation

| POD | Node | Loopback1 |
| --- | ---- | --------- |
| FABRIC | dc1-leaf1a | 10.255.11.13/32 |
| FABRIC | dc1-leaf1b | 10.255.11.13/32 |
| FABRIC | dc2-leaf1a | 10.255.21.23/32 |
| FABRIC | dc2-leaf1b | 10.255.21.23/32 |

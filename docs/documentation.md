# Technical Documentation — Simple Inter-VLAN Routing

## 1. Introduction

This document describes the design, configuration, operation, and verification of the Simple Inter-VLAN Routing lab implemented in Cisco Packet Tracer.

The lab uses a Cisco router with three physical interfaces connected to a Layer 2 switch. Each interface serves as the default gateway for a separate VLAN and IPv4 subnet.

The objective is to understand how Layer 2 segmentation and Layer 3 routing work together to enable communication between logically separated networks.

## 2. Topology and Architecture

The lab contains:

- **R1:** Cisco router responsible for inter-VLAN routing.
- **SW1:** Layer 2 switch responsible for VLAN segmentation and frame switching.
- **Six PCs:** End devices distributed across three VLANs.
- **Three VLANs:** HR, IT, and SALES.

### Design principle

Each VLAN has a dedicated physical connection to the router through a switch access port.

```text
VLAN 10 — HR ───── SW1 access port ───── R1 G0/0
VLAN 20 — IT ───── SW1 access port ───── R1 G0/1
VLAN 30 — SALES ── SW1 access port ───── R1 G0/2
```

The diagram above represents the logical relationship between the VLANs and router interfaces. Refer to `topology.png` for the actual physical topology and device connections.

Unlike Router-on-a-Stick, this design does not require an 802.1Q trunk between SW1 and R1, nor does it require router subinterfaces.

## 3. VLAN Configuration

Three VLANs are created on SW1 to separate the devices into departmental broadcast domains.

| VLAN ID | Name | Purpose |
|---|---|---|
| 10 | HR | Human Resources network |
| 20 | IT | Information Technology network |
| 30 | SALES | Sales network |

### Access-port assignment

Each end-device port and router-facing port is assigned to the appropriate VLAN.

**VLAN 10 — HR** --> interfaces fa3/1,fa4/1,g0/1
**VLAN 20 — IT** --> interfaces fa5/1,fa6/1,g1/1
**VLAN 30 — SALES** --> interfaces fa7/1,fa8/1,g2/1

These interface identifiers correspond to the switch model and port numbering used in the simulation.

The `switchport mode access` command forces each selected port to operate as an access port. The `switchport access vlan` command associates the port with the designated VLAN.

This is an important distinction: the router-facing switch ports are access ports, not trunk ports.

## 4. IPv4 Addressing and Subnetting

The lab uses three `/26` networks derived from the `10.0.0.0/24` address space.

A `/26` prefix corresponds to the subnet mask `255.255.255.192`, providing 64 addresses per subnet, including the network and broadcast addresses, and 62 usable host addresses.

| Parameter | VLAN 10 | VLAN 20 | VLAN 30 |
|---|---|---|---|
| Network | `10.0.0.0/26` | `10.0.0.64/26` | `10.0.0.128/26` |
| First usable | `10.0.0.1` | `10.0.0.65` | `10.0.0.129` |
| Last usable | `10.0.0.62` | `10.0.0.126` | `10.0.0.190` |
| Broadcast | `10.0.0.63` | `10.0.0.127` | `10.0.0.191` |
| Default gateway | `10.0.0.62` | `10.0.0.126` | `10.0.0.190` |

The last usable address in each subnet is reserved for the corresponding router interface.

Each PC must have:
- An IPv4 address belonging to its VLAN's subnet.
- The subnet mask `255.255.255.192`.
- The default gateway corresponding to its VLAN.

Host addresses must be unique and must not use the network, broadcast, or router gateway address.

## 5. Router Configuration

R1 uses three physical Gigabit Ethernet interfaces to provide Layer 3 connectivity for the three VLANs.

### Interface configuration

```text
interface g0/0
 description ## GATEWAY FOR VLAN 10 ##
 ip address 10.0.0.62 255.255.255.192
 no shutdown
```

### Explanation

- `interface`: selects the physical router interface.
- `description`: documents the interface's intended role.
- `ip address`: assigns the interface its IPv4 address and subnet mask.
- `no shutdown`: enables the interface administratively.

No static routes are required for communication between these three networks. Once the interfaces are configured, enabled, and operational, Cisco IOS automatically installs routes for the directly connected networks.

## 6. How Inter-VLAN Routing Works

Consider a host in VLAN 10 communicating with a host in VLAN 20.

1. The source host determines that the destination IP address belongs to a different subnet.
2. It sends the packet to its default gateway, `10.0.0.62`.
3. SW1 forwards the Ethernet frame through the access port assigned to VLAN 10 and delivers it to R1.
4. R1 examines the destination IP address and consults its routing table.
5. R1 forwards the packet through `GigabitEthernet0/1`, which connects to the VLAN 20 network.
6. SW1 receives the frame through its VLAN 20 access port and forwards it to the destination host.

The reverse traffic follows the corresponding path back through R1.

The router performs Layer 3 forwarding between IP networks, while the switch handles Layer 2 forwarding within each VLAN.

## 7. Verification and Testing

Verification is performed at both Layer 2 and Layer 3.

### 7.1 Verify router interfaces

```text
R1# show ip interface brief
```

Expected result: `GigabitEthernet0/0`, `GigabitEthernet0/1`, and `GigabitEthernet0/2` display their configured addresses and have `up/up` status.

The recorded output in `tests/shows.txt` confirms this state.

### 7.2 Verify the routing table

```text
R1# show ip route
```

The routing table should contain these directly connected networks:

```text
C 10.0.0.0/26 is directly connected, GigabitEthernet0/0
C 10.0.0.64/26 is directly connected, GigabitEthernet0/1
C 10.0.0.128/26 is directly connected, GigabitEthernet0/2
```

`C` denotes a directly connected route. `L` denotes a local route for the router interface's own IPv4 address.

The recorded output confirms that all three subnets are present. A default route is not required for communication between these directly connected networks.

### 7.3 Verify VLAN membership

```text
SW1# show vlan brief
```

Confirm that VLANs 10, 20, and 30 are active and that the intended access ports belong to the correct VLANs.

### 7.4 Verify trunk status

```text
SW1# show interfaces trunk
```

No trunk is required in this design. An empty trunk output is therefore consistent with the intended configuration, provided the access links and VLAN assignments are correct.

### 7.5 Verify end-to-end connectivity

From a PC, open the command prompt and test connectivity using `ping`.

For example, test a host in VLAN 10 against a host in VLAN 20, then test communication with a host in VLAN 30.

Also verify same-VLAN connectivity to distinguish Layer 2 issues from inter-VLAN routing issues.

The repository includes the connectivity screenshot in `tests/end-to-end-connectivity.png` and simulation evidence in `tests/simulation-mode.png`.

## 8. Troubleshooting Checklist

If connectivity fails, verify the following in order:

1. **Physical connectivity:** Confirm that the relevant links are connected and operational.
2. **VLAN membership:** Check that each PC and router-facing switch port belongs to the intended VLAN.
3. **Router interface status:** Verify the IP addresses and `up/up` state of all three router interfaces.
4. **Host addressing:** Check the IPv4 address, subnet mask, and default gateway on each PC.
5. **Routing table:** Confirm that all three connected routes are present on R1.
6. **ICMP tests:** Test the default gateway first, then a host in the same VLAN, and finally hosts in other VLANs.

A failed ping alone does not identify the root cause. Troubleshooting should proceed from physical and Layer 2 connectivity toward Layer 3 routing and host configuration.

## 9. Architectural Considerations

### Advantages

- Straightforward configuration and troubleshooting.
- Clear mapping between physical router interfaces and VLANs.
- No router subinterfaces or router-facing trunk configuration required.
- Useful for understanding the relationship between VLANs, IPv4 subnets, gateways, and routing tables.

### Limitations

- Each VLAN requires a dedicated physical router interface.
- Router interface availability limits the number of VLANs supported.
- Additional VLANs require additional physical interfaces and switch connections.
- The design can consume more cabling and router ports than alternative approaches.

For larger networks, Router-on-a-Stick can route multiple VLANs over a single physical router interface using 802.1Q subinterfaces. A multilayer switch can instead route between VLANs using switched virtual interfaces (SVIs).

## 10. Conclusion

This lab demonstrates a basic but important inter-VLAN routing architecture. By combining access-port VLAN assignments, dedicated router interfaces, IPv4 subnetting, and default gateways, devices in separate broadcast domains can communicate through Layer 3 routing.

The configuration and verification evidence provide a practical foundation for understanding more scalable designs, including Router-on-a-Stick and multilayer switching.

## Author

**C. Victorin Vodinou**

[GitHub Profile](https://github.com/Victorin13)
[LinkedIn Profile](https://www.linkedin.com/in/victorin-vodinou/)
Gmail Prifile: victorinvodinou@gmail.com

Networking | Systems Administration | Network Security
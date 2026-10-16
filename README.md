# Inter-VLAN Routing — Physical Interfaces

## Overview

This lab demonstrates a basic implementation of Inter-VLAN Routing
using a router with multiple physical interfaces.

Each VLAN is associated with a dedicated physical router interface,
allowing the router to route traffic between the different VLANs.

## Topology

The topology consists of:

- 1 Cisco router
- 1 Cisco Catalyst switch
- 6 PCs
- 3 VLANs

## VLANs

| VLAN | Department | Network   | Prefix |
|------|------------|-----------|--------|
| 10   | IT         | 10.0.0.0  | /26    |
| 20   | HR         | 10.0.0.64 | /26    |
| 30   | SALES      | 10.0.0.128| /26    |

## Objectives

- Create and configure VLANs
- Assign switch ports to VLANs
- Configure router interfaces
- Configure IPv4 addressing
- Configure default gateways
- Enable Inter-VLAN Routing
- Verify connectivity between VLANs

## Key Concept

In this architecture, each VLAN requires a dedicated physical
router interface.

This approach is simple but does not scale efficiently as the
number of VLANs increases.

## Validation

Connectivity was verified using ICMP ping between hosts belonging
to different VLANs.

## Repository Structure

```text
.
├── configs/
│   └── Device configuration files (.txt)
├── docs/
│   ├── documentation.md
│   └── topology.png
├── src/
│   └── VTP-DTP-Lab.pkt
├── tests/
│   ├── simulation-mode.png
│   ├── end-to-end-connectivity.png
│   ├── shows.txt
│   └── Additional verification screenshots
├── .gitignore
├── LICENSE
└── README.md
```
## Author

**C. Victorin Vodinou**

[GitHub Profile](https://github.com/Victorin13)
[LinkedIn Profile](https://www.linkedin.com/in/victorin-vodinou/)
Gmail Prifile: victorinvodinou@gmail.com

Networking | Systems Administration | Network Security
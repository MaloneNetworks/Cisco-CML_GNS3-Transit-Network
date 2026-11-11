# Cisco CML–GNS3 Transit Network

A small lab build showing how I connected Cisco Modeling Labs (CML) and GNS3 through a shared VMware LAN Segment.

The goal was to establish a working connection between the two lab environments, verify router-to-router communication, and document the setup so the transit can be reused with the main enterprise/VoIP lab.

## What this build does

- Connects a Cisco CML lab to a GNS3 lab through VMware Workstation.
- Keeps VM management traffic separate from the lab transit connection.
- Uses a dedicated Cisco CML bridge and a GNS3 Cloud node to connect the two environments.
- Confirms end-to-end transit connectivity with successful pings in both directions.

## Lab overview

| Component | Interface / setting | Address or role |
|---|---|---|
| Cisco CML VM | Management: `bridge0` / `ens160` | `192.168.4.82/22` |
| GNS3 VM | Management: `eth0` | `192.168.4.79/22` |
| Cisco CML VM | Lab adapter: `ens32` | VMware LAN Segment: `VOIP-LAB` |
| Cisco CML VM | Dedicated bridge | `bridge1`, member: `ens32` |
| GNS3 VM | Lab adapter: `eth1` | VMware LAN Segment: `VOIP-LAB` |
| Cisco CML lab | External Connector | `bridge1` |
| GNS3 lab | Cloud node | `Cloud2`, bound to `eth1` |

The VM management network remained on `192.168.4.0/22`. The lab transit used a separate point-to-point network:

| Router | Interface | Transit address |
|---|---|---|
| GNS3 R1 | FastEthernet0/0 | `10.255.255.1/30` |
| Cisco CML IOL-0 | Ethernet0/0 | `10.255.255.2/30` |

## Connectivity verification

The transit connection was tested from both lab routers. Each router successfully pinged the other across the `10.255.255.0/30` link, with **5 out of 5 replies in each direction (100% success)**.

![Successful bidirectional pings between Cisco CML and GNS3](successful-cml-gns3-pings.png)

*Figure 1. Successful router-to-router ping tests from Cisco CML to GNS3 and from GNS3 to Cisco CML.*

## Documentation

See [TRANSIT-BUILD.md](TRANSIT-BUILD.md) for the adapter mapping, bridge setup, router configuration, verification steps, and troubleshooting notes.

This repository documents the completed transit build and its successful connectivity test. The connection is ready to be referenced when integrating Cisco CML devices or services with the separate enterprise/VoIP lab.

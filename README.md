# CML–GNS3 Transit Network

A small lab build showing how I connected Cisco Modeling Labs (CML) and GNS3 through a shared VMware LAN Segment.

The goal was to get a router in CML talking to a router in GNS3, then document the working transit path so it can be reused with my main enterprise/VoIP lab.

## What this build does

- Connects a CML lab to a GNS3 lab through VMware Workstation.
- Keeps VM management traffic separate from the lab transit connection.
- Uses a dedicated CML bridge and a GNS3 Cloud node to connect the two environments.
- Verifies connectivity with a successful router-to-router ping.

## Lab overview

| Component | Interface / setting | Address or role |
|---|---|---|
| CML VM | Management: `bridge0` / `ens160` | `192.168.4.82/22` |
| GNS3 VM | Management: `eth0` | `192.168.4.79/22` |
| CML VM | Lab adapter: `ens32` | VMware LAN Segment: `VOIP-LAB` |
| CML VM | Dedicated bridge | `bridge1`, member: `ens32` |
| GNS3 VM | Lab adapter: `eth1` | VMware LAN Segment: `VOIP-LAB` |
| CML lab | External Connector | `bridge1` |
| GNS3 lab | Cloud node | `Cloud2`, bound to `eth1` |

The VM management network remained on `192.168.4.0/22`. The lab transit used a separate point-to-point network:

| Router | Interface | Transit address |
|---|---|---|
| GNS3 R1 | FastEthernet0/0 | `10.255.255.1/30` |
| CML IOL-0 | Ethernet0/0 | `10.255.255.2/30` |

## Result

**Connectivity was verified in both directions.** The two lab routers successfully pinged each other across the CML–GNS3 transit connection.

## Documentation

See [TRANSIT-BUILD.md](TRANSIT-BUILD.md) for the adapter mapping, bridge setup, router configuration, verification steps, and troubleshooting notes.

This repository records the completed transit build. The connection can be referenced when bringing CML services or devices into the separate enterprise/VoIP lab.

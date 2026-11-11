# Cisco CML–GNS3 Transit Build

## 1. Purpose

This build connects Cisco Modeling Labs (CML) and GNS3 so devices in either lab can communicate across a shared Layer 2 path.

The work was done in VMware Workstation on one host. The completed test used one router in Cisco CML and one router in GNS3. The purpose was to prove the transit connection before using it with the larger enterprise/VoIP lab.

## 2. Design

Both virtual machines have two network adapters:

- **Adapter 1 — Bridged:** VM management access on the management network.
- **Adapter 2 — LAN Segment `VOIP-LAB`:** A separate Layer 2 segment used for lab-to-lab traffic.

The management network and the lab transit are separate. The management adapters were not used as the lab transit path.

### VMware and Linux interface mapping

| VM | VMware adapter | VMware network | Linux interface | MAC ending |
|---|---|---|---|---|
| Cisco CML VM | Adapter 1 | Bridged | `ens160` | `d1` |
| Cisco CML VM | Adapter 2 | LAN Segment: `VOIP-LAB` | `ens32` | `db` |
| GNS3 VM | Adapter 1 | Bridged | `eth0` | `8d` |
| GNS3 VM | Adapter 2 | LAN Segment: `VOIP-LAB` | `eth1` | `97` |

### Management addressing

| VM | Management interface | Address | Default gateway |
|---|---|---|---|
| Cisco CML VM | `bridge0` | `192.168.4.82/22` | `192.168.4.1` |
| GNS3 VM | `eth0` | `192.168.4.79/22` | `192.168.4.1` |

Management connectivity was checked in both directions with ping. Both tests received four replies with no packet loss.

### Lab transit addressing

The router-to-router test used `10.255.255.0/30`.

| Device | Interface | Address |
|---|---|---|
| GNS3 R1 | FastEthernet0/0 | `10.255.255.1/30` |
| Cisco CML IOL-0 | Ethernet0/0 | `10.255.255.2/30` |

The /30 network provides two usable addresses for this point-to-point test.

## 3. Cisco CML bridge setup

Cisco CML was running version **2.10.0**.

The second Cisco CML VM adapter, `ens32`, was connected to VMware's `VOIP-LAB` LAN Segment. A dedicated bridge was created for that adapter. The existing management bridge, `bridge0`, was left unchanged.

### Bridge settings

A bridge named `bridge1` was created in Cisco CML System Administration under **Networking**.

The bridge was configured with:

- IPv4: Disabled
- IPv6: Disabled
- Connect automatically: Enabled
- Interface member: `ens32`

The bridge member list showed `ens32`, and the bridge reported carrier present.

The new bridge was then made available to Cisco CML Workbench through the **External Connectors** administration page. In the Cisco CML topology, the External Connector node was set to use `bridge1`.

> Keep the management bridge and management adapter separate. Do not attach `ens160` to the lab bridge.

## 4. GNS3 Cloud setup

A Cloud node named `Cloud2` was added to the GNS3 project.

In the Cloud node configuration:

1. Open the **Ethernet interfaces** tab.
2. Select `eth1`.
3. Add the interface and apply the change.
4. Connect Cloud2 to the test router's FastEthernet0/0 interface.

The Cloud node was configured to use the GNS3 VM interface, not a local host interface. The GNS3 VM's `eth1` was the adapter connected to the same VMware `VOIP-LAB` segment as Cisco CML's `ens32`.

## 5. Router configuration

The following configuration was used for the transit test. Interface names match the devices used in this build.

### GNS3 R1

```text
enable
configure terminal
interface FastEthernet0/0
 description Cisco CML-GNS3-VOIP-LAB-TEST
 ip address 10.255.255.1 255.255.255.252
 no shutdown
end
```

### Cisco CML IOL-0

```text
enable
configure terminal
interface Ethernet0/0
 description GNS3-Cisco CML-EXTERNAL-TEST
 ip address 10.255.255.2 255.255.255.252
 no shutdown
end
```

No default routes or dynamic routing protocols were needed for this directly connected /30 test.

## 6. Verification

### 6.1 Check VM interfaces

On the Cisco CML VM:

```bash
ip -br addr
ip -br link
bridge link
```

Expected lab interface mapping:

- `ens32` is the second adapter attached to `VOIP-LAB`.
- `ens32` is a member of `bridge1`.
- `bridge0` remains the management bridge on `ens160`.

On the GNS3 VM:

```bash
ip -br addr
ip link show eth1
```

Expected lab interface mapping:

- `eth1` is UP.
- `eth1` is the second adapter attached to `VOIP-LAB`.

### 6.2 Check router interfaces

On GNS3 R1:

```text
show ip interface brief
```

On Cisco CML IOL-0:

```text
show ip interface brief
```

Both transit interfaces should show the configured address and an operational status of `up/up`.

### 6.3 Test router-to-router connectivity

From GNS3 R1:

```text
ping 10.255.255.2
```

From Cisco CML IOL-0:

```text
ping 10.255.255.1
```

Both directions were successfully tested.

### 6.4 ARP and packet capture checks

During troubleshooting, the GNS3 VM was used to capture ARP traffic on `eth1`:

```bash
sudo tcpdump -ni eth1 arp
```

The capture showed GNS3 R1 sending ARP requests for `10.255.255.2`. At first, no reply was received because the Cisco CML External Connector node had not been started. Starting the External Connector allowed traffic to pass, and the router ping succeeded.

## 7. Troubleshooting notes

### Router ping fails and ARP is incomplete

Check the following:

1. Both VMware second adapters use the exact same LAN Segment name: `VOIP-LAB`.
2. Cisco CML's `ens32` is attached to `bridge1`.
3. The Cisco CML External Connector is configured to use `bridge1`.
4. The Cisco CML External Connector node is started.
5. GNS3 Cloud2 has `eth1` assigned.
6. Cloud2 is connected to the intended GNS3 router interface.
7. Both router interfaces are enabled and use addresses in the same /30 network.

A green link indicator alone does not prove end-to-end traffic is passing. Verify with ARP and ping.

## 8. Final verified state

The completed test established:

- Cisco CML and GNS3 VM management interfaces can reach each other on `192.168.4.0/22`.
- The Cisco CML VM's `ens32` is connected to `bridge1`.
- Cisco CML's External Connector uses `bridge1`.
- The GNS3 Cloud node uses GNS3 VM `eth1`.
- Both lab routers have working transit interface configurations.
- Router-to-router ICMP connectivity works in both directions across `10.255.255.0/30`.

This proves the basic Cisco CML-to-GNS3 transit path. It does not, by itself, prove that every service, VLAN, or device in either full lab is reachable. Those paths depend on the routing, firewall, and switching configuration built on top of this transit.

## 9. Reference

This transit build is a standalone lab record. It can be used as the connection reference when integrating Cisco CML-based services and devices with the separate enterprise/VoIP GNS3 project.

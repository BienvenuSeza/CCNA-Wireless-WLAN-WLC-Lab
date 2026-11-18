# Enterprise Campus Network: Inter-VLAN Routing, DHCP Relay and WLC Architecture

## 1. Project Overview

This project documents an enterprise campus network architecture simulated in **Cisco Packet Tracer**.

The network demonstrates Layer 3 Inter-VLAN Routing, centralized DHCP services, DHCP Relay, VLAN segmentation, and Wireless LAN Controller (WLC) integration.

A Cisco Catalyst 3560 Multilayer Switch provides routing between the management and wireless client VLANs. A centralized server provides DHCP and DNS services, while the WLC manages lightweight wireless access points.

### Technologies implemented

- Cisco Catalyst 3560 Multilayer Switch
- VLAN segmentation
- Inter-VLAN Routing using Switch Virtual Interfaces (SVIs)
- DHCP Server
- DHCP Relay using `ip helper-address`
- Cisco Wireless LAN Controller 3504
- Lightweight Access Points
- CAPWAP
- Wireless client connectivity

---

## 2. Network Devices and Roles

| Device | Model or Type | Role |
|---|---|---|
| SW-Colmar | Cisco Catalyst 3560 | Layer 2 and Layer 3 switching |
| WLC | Cisco Wireless LAN Controller 3504 | Centralized wireless management |
| Server0 | Server | DHCP and DNS services |
| LAP0 | Lightweight Access Point | Wireless access |
| LAP1 | Lightweight Access Point | Wireless access |
| Laptop0 | Laptop | Client connectivity |
| PC0 | PC | Wired client connectivity |
| Smartphone0 | Smartphone | Wireless client connectivity |

---

## 3. VLAN and IP Addressing Plan

The network uses two VLANs to separate infrastructure management from wireless client traffic.

### 3.1 VLAN configuration

| VLAN ID | VLAN Name | Network Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| 10 | MANAGEMENT_SERVEUR | 10.0.0.0/8 | 255.0.0.0 | 10.0.0.1 |
| 20 | CLIENTS_WIFI | 20.0.0.0/8 | 255.0.0.0 | 20.0.0.1 |

### 3.2 Infrastructure IP addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| SW-Colmar | VLAN 10 | 10.0.0.1 | 255.0.0.0 |
| SW-Colmar | VLAN 20 | 20.0.0.1 | 255.0.0.0 |
| Server0 | FastEthernet | 10.0.0.10 | 255.0.0.0 |
| WLC | Management | 10.0.0.11 | 255.0.0.0 |

**Addressing note:** `10.0.0.0/8` is private IPv4 address space. `20.0.0.0/8` is public IPv4 address space, not private. It is retained here to document the actual Packet Tracer lab addressing. For a production enterprise network, use RFC 1918 private addressing for both VLANs.

---

## 4. Physical Network Connections

The following table describes the switch port allocation in the reference topology.

| Switch Port | Connected Device | Port Mode | VLAN |
|---|---|---|---|
| Fa0/1 | WLC | Trunk | 10, 20 |
| Fa0/3 | Laptop0 | Access | 20 |
| Fa0/4 | LAP1 | Access | 10 |
| Fa0/5 | LAP0 | Access | 10 |
| Fa0/6 | Server0 | Access | 10 |
| Fa0/16 | PC0 | Access | 20 |

The trunk connection between the switch and WLC carries VLAN 10 and VLAN 20 traffic.

The lightweight access points connect to the management VLAN.

---

## 5. Catalyst 3560 Configuration

### 5.1 Create VLANs

```cisco
enable
configure terminal

vlan 10
 name MANAGEMENT_SERVEUR
exit

vlan 20
 name CLIENTS_WIFI
exit
```

### 5.2 Configure the WLC trunk port

```cisco
interface FastEthernet0/1
 description TRUNK_TO_WLC
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10,20
 no shutdown
exit
```

If the Packet Tracer switch does not support `switchport trunk encapsulation dot1q`, omit that command. Some switch models use 802.1Q without requiring explicit encapsulation configuration.

### 5.3 Configure access ports

```cisco
interface FastEthernet0/3
 description LAPTOP0_CLIENT
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

interface range FastEthernet0/4 - 5
 description LIGHTWEIGHT_ACCESS_POINTS
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface FastEthernet0/6
 description SERVER0_DHCP_DNS
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface FastEthernet0/16
 description PC0_CLIENT
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### 5.4 Enable Layer 3 routing

```cisco
ip routing
```

This command enables IPv4 routing between the VLAN interfaces on the multilayer switch.

### 5.5 Configure Switch Virtual Interfaces

VLAN 10 provides the management gateway, while VLAN 20 provides the wireless client gateway.

```cisco
interface vlan 10
 description MANAGEMENT_GATEWAY
 ip address 10.0.0.1 255.0.0.0
 no shutdown
exit

interface vlan 20
 description WIFI_CLIENT_GATEWAY
 ip address 20.0.0.1 255.0.0.0
 ip helper-address 10.0.0.10
 no shutdown
exit
```

The `ip helper-address` command forwards DHCP broadcast requests received in VLAN 20 to the DHCP server at `10.0.0.10`.

This allows clients in VLAN 20 to obtain IP addresses from a DHCP server located in VLAN 10.

---

## 6. Server0 Configuration

### 6.1 Static IPv4 configuration

Configure Server0 with the following network parameters.

| Parameter | Value |
|---|---|
| IPv4 Address | 10.0.0.10 |
| Subnet Mask | 255.0.0.0 |
| Default Gateway | 10.0.0.1 |
| DNS Server | 10.0.0.10 |

### 6.2 DHCP pool for VLAN 10

Enable the DHCP service on Server0 and create the following pool.

| Parameter | Value |
|---|---|
| Pool Name | MANAGEMENT_POOL |
| Default Gateway | 10.0.0.1 |
| DNS Server | 10.0.0.10 |
| Start IP Address | 10.0.0.101 |
| Subnet Mask | 255.0.0.0 |
| Maximum Users | 150 |

### 6.3 DHCP pool for VLAN 20

Create a second DHCP pool for wireless clients.

| Parameter | Value |
|---|---|
| Pool Name | WIFI_CLIENTS_POOL |
| Default Gateway | 20.0.0.1 |
| DNS Server | 10.0.0.10 |
| Start IP Address | 20.0.0.101 |
| Subnet Mask | 255.0.0.0 |
| Maximum Users | 150 |

### 6.4 DHCP Relay operation

The DHCP server is connected to VLAN 10, while wireless clients belong to VLAN 20.

DHCP Discover messages are initially broadcast messages. Routers do not forward these broadcasts by default.

The `ip helper-address 10.0.0.10` command on the VLAN 20 SVI allows the multilayer switch to relay DHCP requests to Server0.

The server can then allocate addresses from the appropriate DHCP pool based on the relay information.

---

## 7. Wireless LAN Controller Configuration

The Cisco 3504 WLC centralizes wireless network management and controls the lightweight access points.

### 7.1 Management interface

Configure the WLC management interface using the following reference settings.

| Parameter | Value |
|---|---|
| Management VLAN | 10 |
| IP Address | 10.0.0.11 |
| Subnet Mask | 255.0.0.0 |
| Default Gateway | 10.0.0.1 |

### 7.2 Dynamic interface for wireless clients

The WLC requires an appropriate dynamic interface associated with the wireless client VLAN.

The following is a reference example. Confirm the actual interface IP and configuration in your Packet Tracer project.

| Parameter | Example Value |
|---|---|
| Interface Name | WIFI_CLIENTS |
| VLAN ID | 20 |
| IP Address | 20.0.0.2 |
| Subnet Mask | 255.0.0.0 |
| Gateway | 20.0.0.1 |
| DHCP Server | 10.0.0.10 |

The dynamic interface maps wireless client traffic to VLAN 20.

### 7.3 WLAN configuration

Create and enable a WLAN on the WLC.

| Parameter | Example Value |
|---|---|
| WLAN Profile | CORPORATE_WIFI |
| SSID | CORPORATE_WIFI |
| VLAN | 20 |
| Security | WPA2-PSK |
| Client Addressing | DHCP |

Use a secure and unique pre-shared key in the actual lab.

---

## 8. Lightweight AP Discovery and CAPWAP

Lightweight Access Points establish a CAPWAP relationship with the WLC.

CAPWAP provides the control and management communication between the WLC and lightweight access points.

### 8.1 AP discovery methods

Access points can discover a WLC through several mechanisms:

- DHCP Option 43.
- DNS discovery using `CISCO-CAPWAP-CONTROLLER`.
- Previously configured controller information.
- Layer 3 discovery mechanisms supported by the platform.

### 8.2 DHCP Option 43

The WLC management IP address is:

`10.0.0.11`

DHCP Option 43 can provide the controller address to lightweight access points.

The hexadecimal encoding is vendor-specific. Verify the correct encoding and Packet Tracer support before configuring this option.

### 8.3 CAPWAP verification

Verify the following:

1. The lightweight AP receives a valid management IP address.
2. The AP can reach the WLC management IP.
3. The WLC discovers and registers the AP.
4. The AP downloads its configuration.
5. The WLAN becomes available to wireless clients.

---

## 9. Network Verification

### 9.1 Switch verification commands

Use the following commands on SW-Colmar.

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show running-config
```

These commands help verify VLAN membership, trunk operation, SVI status, routing, and the active configuration.

### 9.2 Connectivity tests

From a client in VLAN 20, test the following destinations.

```text
ping 20.0.0.1
ping 10.0.0.10
ping 10.0.0.11
```

Expected results:

- The first test checks connectivity to the VLAN 20 default gateway.
- The second test checks connectivity to the DHCP and DNS server.
- The third test checks connectivity to the WLC management interface.

### 9.3 DHCP verification

On a Windows client, execute:

```powershell
ipconfig /all
ipconfig /release
ipconfig /renew
```

Verify that the client receives:

- An IPv4 address from the correct DHCP pool.
- The correct subnet mask.
- The correct default gateway.
- The DNS server address `10.0.0.10`.

### 9.4 Wireless verification

Verify the following:

- The WLAN is enabled on the WLC.
- The lightweight APs are registered.
- The wireless client can detect the SSID.
- The client can authenticate successfully.
- The client receives an IP address from VLAN 20.
- The client can reach its default gateway.

---

## 10. Troubleshooting Scenarios

| Symptom | Possible Cause | Verification |
|---|---|---|
| Client receives 169.254.x.x | DHCP failure | `ipconfig /all` |
| Client cannot reach its gateway | VLAN or SVI issue | `show vlan brief` and `show ip interface brief` |
| DHCP works in VLAN 10 but not VLAN 20 | Missing or incorrect DHCP Relay | Check `ip helper-address` |
| AP does not register with WLC | CAPWAP discovery or connectivity issue | Check AP IP configuration and WLC reachability |
| WLAN is visible but client cannot connect | Security or WLAN configuration mismatch | Check SSID and security settings |
| Client receives an incorrect subnet | Incorrect DHCP pool or VLAN assignment | Check DHCP configuration and switch port |

---

## 11. CCNA 200-301 Learning Objectives

This lab demonstrates practical knowledge of:

- VLAN creation and assignment.
- Access ports and 802.1Q trunk ports.
- Layer 3 switching.
- Inter-VLAN Routing.
- Switch Virtual Interfaces.
- DHCP services.
- DHCP Relay using `ip helper-address`.
- Wireless LAN Controller architecture.
- Lightweight AP discovery.
- CAPWAP.
- Wired and wireless client troubleshooting.

---

## 12. Project Status

**Project:** Enterprise Campus Network

**Platform:** Cisco Packet Tracer

**Documentation Version:** 1.0

**Status:** Architecture documented. Configuration and end-to-end verification should be confirmed against the actual Packet Tracer project.

Future documentation will include device configurations, topology screenshots, DHCP lease evidence, WLC interface screenshots, and connectivity test results.
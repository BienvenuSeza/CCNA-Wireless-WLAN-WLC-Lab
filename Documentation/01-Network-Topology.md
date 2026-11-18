# 01 - Network Topology



## 1. Project Overview



This lab implements a centralized wireless network using a Cisco 3504 Wireless LAN Controller (WLC), lightweight access points, and a Cisco Catalyst 2960 switch.



The lab is designed to develop practical skills related to wireless networking, WLAN management, and network troubleshooting for the Cisco CCNA 200-301 certification.



## 2. Network Devices



| Device | Model / Type | Role |

|---|---|---|

| WLC-3504 | Cisco 3504 | Centralized wireless management |

| Switch0 | Cisco Catalyst 2960 | Layer 2 network connectivity |

| Server0 | Server | Network services |

| Laptop0 | Laptop | Wired network client |

| LAP0 | Lightweight AP | Wireless access |

| LAP1 | Lightweight AP | Wireless access |

| Smartphone0 | Smartphone | Wireless client |



## 3. Physical Connections



The devices are connected to the Catalyst 2960 switch as follows:



| Switch Interface | Connected Device |

|---|---|

| Fa0/1 | WLC-3504 |

| Fa0/2 | Server0 |

| Fa0/3 | Laptop0 |

| Fa0/4 | LAP1 |

| Fa0/5 | LAP0 |



Smartphone0 connects wirelessly through an access point.



## 4. Network Architecture



The Catalyst 2960 provides Layer 2 connectivity between the network devices.



The Cisco 3504 WLC provides centralized management of the lightweight access points.



The lightweight access points provide wireless connectivity to wireless clients.



## 5. Lab Objectives



\- Understand a controller-based wireless architecture.

\- Establish connectivity between the WLC and lightweight access points.

\- Configure WLAN and SSID settings.

\- Implement wireless security.

\- Connect wireless clients.

\- Verify wired and wireless connectivity.

\- Troubleshoot network communication issues.



## 6. Network Topology Screenshot



See the complete network topology:



`../Screenshots/01-network-topology.png`



## 7. Project Status



Initial topology documented. Device configuration and connectivity verification will be documented in subsequent sections.



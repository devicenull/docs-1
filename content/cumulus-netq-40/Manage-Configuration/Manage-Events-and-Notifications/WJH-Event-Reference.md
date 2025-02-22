---
title: WJH Event Messages Reference
author: NVIDIA
weight: 813
toc: 4
---
This reference lists all the NetQ-supported WJH metrics and provides a brief description of each. The full outputs vary slightly based on the type of drop and whether you are viewing the results in the NetQ UI or through one of the NetQ CLI commands.

For instructions on how to configure and monitor What Just Happened events, refer to {{<link title="Configure and Monitor What Just Happened">}}.

## ACL Drops

Displays the reason why an ACL has dropped packets.

| Reason | Description|
| --- | --- |
| Ingress port ACL | ACL action set to deny on the physical ingress port or bond |
| Ingress router ACL | ACL action set to deny on the ingress switch virtual interfaces (SVIs) |
| Egress port ACL | ACL action set to deny on the physical egress port or bond |
| Egress router ACL | ACL action set to deny on the egress SVIs |

## Buffer Drops

Displays the reason why the server buffer has dropped packets.

<!-- vale off -->
| Reason | Description|
| --- | --- |
| Tail drop | Tail drop is enabled, and buffer queue is filled to maximum capacity |
| WRED | Weighted Random Early Detection is enabled, and buffer queue is filled to maximum capacity or the RED engine dropped the packet as of random congestion prevention |
| Port TC Congestion Threshold Crossed | Percentage of the occupancy buffer exceeded or dropped below the specified high or low threshold |
| Packet Latency Threshold Crossed | Time a packet spent within the switch exceeded or dropped below the specified high or low threshold |

## Layer 1 Drops

Displays the reason why a port is in the down state.

| Reason | Description|
| --- | --- |
| Port admin down | Port has been purposely set down by user |
| Auto-negotiation failure | Negotiation of port speed with peer has failed |
| Logical mismatch with peer link | Logical mismatch with peer link |
| Link training failure | Link is not able to go operational up due to link training failure |
| Peer is sending remote faults | Peer node is not operating correctly |
| Bad signal integrity | Integrity of the signal on port is not sufficient for good communication |
| Cable/transceiver is not supported | The attached cable or transceiver is not supported by this port |
| Cable/transceiver is unplugged | A cable or transceiver is missing or not fully inserted into the port |
| Calibration failure | Calibration failure |
| Port state changes counter | Cumulative number of state changes |
| Symbol error counter | Cumulative number of symbol errors |
| CRC error counter | Cumulative number of CRC errors |

In addition to the reason, the information provided for these drops includes:

| Parameter | Description |
| --- | --- |
| Corrective Action | Provides recommend actions to take to resolve the port down state |
| First Timestamp | Date and time this port was marked as down for the first time |
| Ingress Port | Port accepting incoming traffic |
| CRC Error Count | Number of CRC errors generated by this port |
| Symbol Error Count | Number of Symbol errors generated by this port |
| State Change Count | Number of state changes that have  occurred on this port |
| OPID | Operation identifier; used for internal purposes |
| Is Port Up | Indicates whether the port is in an Up (true) or Down (false) state |

## Layer 2 Drops

Displays the reason for a link to be down.

| Reason | Description|
| --- | --- |
| MLAG port isolation | Not supported for port isolation implemented with system ACL |
| Destination MAC is reserved (DMAC=01-80-C2-00-00-0x) | The address cannot be used by this link |
| VLAN tagging mismatch | VLAN tags on the source and destination do not match |
| Ingress VLAN filtering | Frames whose port is not a member of the VLAN are discarded |
| Ingress spanning tree filter | Port is in Spanning Tree blocking state |
| Unicast MAC table action discard | Currently not supported |
| Multicast egress port list is empty | No ports are defined for multicast egress |
| Port loopback filter | Port is operating in loopback mode; packets are being sent to itself (source MAC address is the same as the destination MAC address |
| Source MAC is multicast | Packets have multicast source MAC address |
| Source MAC equals destination MAC | Source MAC address is the same as the destination MAC address |

In addition to the reason, the information provided for these drops includes:

| Parameter | Description|
| --- | --- |
| Source Port | Port ID where the link originates |
| Source IP | Port IP address where the link originates |
| Source MAC | Port MAC address where the link originates |
| Destination Port | Port ID where the link terminates |
| Destination IP | Port IP address where the link terminates |
| Destination MAC | Port MAC address where the link terminates |
| First Timestamp | Date and time this link was marked as down for the first time |
| Aggregate Count  | Total number of dropped packets |
| Protocol | ID of the communication protocol running on this link |
| Ingress Port | Port accepting incoming traffic |
| OPID | Operation identifier; used for internal purposes |
<!-- vale on -->

## Router Drops

Displays the reason why the server is unable to route a packet.

<!-- vale off -->
| Reason | Description|
| --- | --- |
| Non-routable packet |  Packet has no route in routing table |
| Blackhole route | Packet received with action equal to discard |
| Unresolved next hop | The next hop in the route is unknown |
| Blackhole ARP/neighbor | Packet received with blackhole adjacency |
| IPv6 destination in multicast scope FFx0:/16 | Packet received with multicast destination address in FFx0:/16 address range |
| IPv6 destination in multicast scope FFx1:/16 | Packet received with multicast destination address in FFx1:/16 address range |
| Non-IP packet | Cannot read packet header because it is not an IP packet |
| Unicast destination IP but non-unicast destination MAC | Cannot read packet with IP unicast address when destination MAC address is not unicast (FF:FF:FF:FF:FF:FF) |
| Destination IP is loopback address | Cannot read packet as destination IP address is a loopback address (dip=>127.0.0.0/8) |
| Source IP is multicast | Cannot read packet as source IP address is a multicast address (ipv4 SIP => 224.0.0.0/4) |
| Source IP is in class E | Cannot read packet as source IP address is a Class E address |
| Source IP is loopback address | Cannot read packet as source IP address is a loopback address ( ipv4 => 127.0.0.0/8 for ipv6 => ::1/128) |
| Source IP is unspecified | Cannot read packet as source IP address is unspecified (ipv4 = 0.0.0.0/32; for ipv6 = ::0) |
| Checksum or IP ver or IPv4 IHL too short | Cannot read packet due to header checksum error, IP version mismatch, or IPv4 header length is too short |
| Multicast MAC mismatch |  For IPv4, destination MAC address is not equal to {0x01-00-5E-0 (25 bits), DIP\[22:0\]} and DIP is multicast. For IPv6, destination MAC address is not equal to {0x3333, DIP\[31:0\]} and DIP is multicast |
| Source IP equals destination IP | Packet has a source IP address equal to the destination IP address |
| IPv4 source IP is limited broadcast | Packet has broadcast source IP address |
| IPv4 destination IP is local network (destination = 0.0.0.0/8) | Packet has IPv4 destination address that is a local network (destination=0.0.0.0/8) |
| IPv4 destination IP is link-local | Packet has IPv4 destination address that is a local link |
| Ingress router interface is disabled | Packet destined to a different subnet cannot be routed because ingress router interface is disabled |
| Egress router interface is disabled | Packet destined to a different subnet cannot be routed because egress router interface is disabled |
| IPv4 routing table (LPM) unicast miss | No route available in routing table for packet |
| IPv6 routing table (LPM) unicast miss | No route available in routing table for packet |
| Router interface loopback | Packet has destination IP address that is local. For example, SIP = 1.1.1.1, DIP = 1.1.1.128. |
| Packet size is larger than MTU | Packet has larger MTU configured than the VLAN |
| TTL value is too small | Packet has TTL value of 1 |
<!-- vale on -->

## Tunnel Drops

Displays the reason for a tunnel to be down.

| Reason | Description|
| --- | --- |
| Overlay switch - source MAC is multicast | Overlay packet's source MAC address is multicast |
| Overlay switch - source MAC equals destination MAC | Overlay packet's source MAC address is the same as the destination MAC address |
| Decapsulation error | De-capsulation produced incorrect format of packet. For example, encapsulation of packet with many VLANs or IP options on the underlay can cause de-capsulation to result in a short packet. |

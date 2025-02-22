---
title: Virtual Router Redundancy - VRR
author: Cumulus Networks
weight: 127
pageID: 8362183
---
Virtual Router Redundancy (VRR) enables hosts to communicate with any
redundant router without reconfiguration, running dynamic router
protocols, or running router redundancy protocols. This means that
redundant routers will respond to Address Resolution Protocol (ARP)
requests from hosts. Routers are configured to respond in an identical
manner, but if one fails, the other redundant routers will continue to
respond, leaving the hosts with the impression that nothing has changed.

Cumulus Linux only supports VRR on switched virtual interfaces (SVIs).
VRR is not supported on physical interfaces or virtual subinterfaces.

The diagram below illustrates a basic VRR-enabled network configuration.
The network includes several hosts, and two routers running Cumulus
Linux configured with [Multi-chassis Link Aggregation](/cumulus-linux-36/Layer-2/Multi-Chassis-Link-Aggregation-MLAG)
(MLAG):

{{< img src="/images/cumulus-linux/vrr-active-active.png" width="600" >}}

{{%notice note%}}

A production implementation will have many more server hosts and network
connections than are shown here. However, this basic configuration
provides a complete description of the important aspects of the VRR
setup.

{{%/notice%}}

As the bridges in each of the redundant routers are connected, they will
each receive and reply to ARP requests for the virtual router IP
address.

{{%notice note%}}

**Multiple ARP Replies**

Each ARP request made by a host will receive replies from each router;
these replies will be identical, and so the host receiving the replies
will either ignore replies after the first, or accept them and overwrite
the previous identical reply, rather than being confused over which
response is correct.

{{%/notice%}}

{{%notice info%}}

**Reserved MAC Address Range**

A range of MAC addresses is reserved for use with VRR, in order to
prevent MAC address conflicts with other interfaces in the same bridged
network. The reserved range is `00:00:5E:00:01:00` to
`00:00:5E:00:01:ff`.

Cumulus Networks recommends using MAC addresses from the reserved range
when configuring VRR.

The reserved MAC address range for VRR is the same as for the Virtual
Router Redundancy Protocol (VRRP), as they serve similar purposes.

{{%/notice%}}

## Configuring a VRR-enabled Network

### Configuring the Routers

The routers implement the layer 2 network interconnecting the hosts and
the redundant routers. To configure the routers, add a bridge with the
following interfaces to each router:

  - One bond interface or switch port interface to each host.
    
    {{%notice note%}}
    
For networks using MLAG, use bond interfaces. Otherwise, use switch
    port interfaces.
    
    {{%/notice%}}

  - One or more interfaces to each peer router.
    
    {{%notice note%}}
    
Multiple inter-peer links are typically bonded interfaces, in order
    to accomodate higher bandwidth between the routers, and to offer
    link redundancy.
    
    {{%/notice%}}
    
    {{%notice note%}}
    
The VLAN interface must have unique IP addresses for both the
    physical (the `address` option below) and virtual (the
    `address-virtual` option below) interfaces, as the unique address is
    used when the switch initiates an ARP request.
    
    {{%/notice%}}

{{%notice info has%}}

**Example VRR Configuration**

The example NCLU commands below create a VLAN-aware bridge interface for
a VRR-enabled network:

    cumulus@switch:~$ net add bridge
    cumulus@switch:~$ net add vlan 500 ip address 192.0.2.252/24
    cumulus@switch:~$ net add vlan 500 ip address-virtual 00:00:5e:00:01:01 192.0.2.254/24
    cumulus@switch:~$ net add vlan 500 ipv6 address 2001:db8::1/32
    cumulus@switch:~$ net add vlan 500 ipv6 address-virtual 00:00:5e:00:01:01 2001:db8::f/32
    cumulus@switch:~$ net pending
    cumulus@switch:~$ net commit

The NCLU commands above produce the following `/etc/network/interfaces`
snippet:

    auto bridge
    iface bridge
        bridge-vids 500
        bridge-vlan-aware yes
     
    auto vlan500
    iface vlan500
        address 192.0.2.252/24
        address 2001:db8::1/32
        address-virtual 00:00:5e:00:01:01 2001:db8::f/32 192.0.2.254/24
        vlan-id 500
        vlan-raw-device bridge

{{%/notice%}}

### Configuring the Hosts

Each host should have two network interfaces. The routers configure the
interfaces as bonds running LACP; the hosts should also configure its
two interfaces using teaming, port aggregation, port group, or
EtherChannel running LACP. Configure the hosts, either statically or via
DHCP, with a gateway address that is the IP address of the virtual
router; this default gateway address never changes.

Configure the links between the hosts and the routers in *active-active*
mode for First Hop Redundancy Protocol.

## Example VRR Configuration with MLAG

To create an
[MLAG](/cumulus-linux-36/Layer-2/Multi-Chassis-Link-Aggregation-MLAG)
configuration that incorporates VRR, use a configuration like the
following:

### leaf01 Configuration

```
cumulus@leaf01:~$ net add interface eth0 ip address 192.168.0.21
cumulus@switch:~$ net add bond server01 bond slaves swp1-2
cumulus@switch:~$ net add bond server01 clag id 1
cumulus@switch:~$ net add bond server01 mtu 9216
cumulus@switch:~$ net add bond server01 alias LACP etherchannel to uplink on server01
cumulus@switch:~$ net add bond peerlink bond slaves swp49-50
cumulus@leaf01:~$ net add interface peerlink.4094 peerlink.4094
cumulus@leaf01:~$ net add interface peerlink.4094 ip address 169.254.255.1/30
cumulus@leaf01:~$ net add interface peerlink.4094 clag peer-ip 169.254.255.2
cumulus@leaf01:~$ net add interface peerlink.4094 clag backup-ip 192.168.0.22
cumulus@leaf01:~$ net add interface peerlink.4094 clag sys-mac 44:38:39:FF:40:90
cumulus@switch:~$ net add bridge bridge ports server01,peerlink
cumulus@switch:~$ net add bridge stp treeprio 4096
cumulus@switch:~$ net add vlan 100 ip address 10.0.1.2/24
cumulus@switch:~$ net add vlan 100 ip address-virtual 44:38:39:FF:00:01 10.0.1.1/24
cumulus@switch:~$ net add vlan 200 ip address 10.0.2.2/24
cumulus@switch:~$ net add vlan 200 ip address-virtual 44:38:39:FF:00:02 10.0.2.1/24
cumulus@switch:~$ net add vlan 300 ip address 10.0.3.2/24
cumulus@switch:~$ net add vlan 300 ip address-virtual 44:38:39:FF:00:03 10.0.3.1/24
cumulus@switch:~$ net add vlan 400 ip address 10.0.4.2/24
cumulus@switch:~$ net add vlan 400 ip address-virtual 44:38:39:FF:00:04 10.0.4.1/24
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

These commands create the following configuration in `/etc/network/interfaces`:

```
auto eth0
iface eth0
    address 192.168.0.21
 
auto bridge
iface bridge
    bridge-ports server01 peerlink
    bridge-vids 100 200 300 400
    bridge-vlan-aware yes
    mstpctl-treeprio 4096
 
auto server01
iface server01
    alias LACP etherchannel to uplink on server01
    bond-slaves swp1 swp2
    clag-id 1
    mtu 9216
 
auto peerlink
iface peerlink
    bond-slaves swp49 swp50
 
auto peerlink.4094
iface peerlink.4094
    address 169.254.255.1/30
    clagd-backup-ip 192.168.0.22
    clagd-peer-ip 169.254.255.2
    clagd-sys-mac 44:38:39:FF:40:90
 
auto vlan100
iface vlan100
    address 10.0.1.2/24
    address-virtual 44:38:39:FF:00:01 10.0.1.1/24
    vlan-id 100
    vlan-raw-device bridge
 
auto vlan200
iface vlan200
    address 10.0.2.2/24
    address-virtual 44:38:39:FF:00:02 10.0.2.1/24
    vlan-id 200
    vlan-raw-device bridge
 
auto vlan300
iface vlan300
    address 10.0.3.2/24
    address-virtual 44:38:39:FF:00:03 10.0.3.1/24
    vlan-id 300
    vlan-raw-device bridge
 
auto vlan400
iface vlan400
    address 10.0.4.2/24
    address-virtual 44:38:39:FF:00:04 10.0.4.1/24
    vlan-id 400
    vlan-raw-device bridge
```

### leaf02 Configuration

```
cumulus@leaf01:~$ net add interface eth0 ip address 192.168.0.22
cumulus@switch:~$ net add bond server01 bond slaves swp1-2
cumulus@switch:~$ net add bond server01 clag id 1
cumulus@switch:~$ net add bond server01 mtu 9216
cumulus@switch:~$ net add bond server01 alias LACP etherchannel to uplink on server01
cumulus@switch:~$ net add bond peerlink bond slaves swp49-50
cumulus@leaf01:~$ net add interface peerlink.4094 peerlink.4094
cumulus@leaf01:~$ net add interface peerlink.4094 ip address 169.254.255.2/30
cumulus@leaf01:~$ net add interface peerlink.4094 clag peer-ip 169.254.255.1
cumulus@leaf01:~$ net add interface peerlink.4094 clag backup-ip 192.168.0.21
cumulus@leaf01:~$ net add interface peerlink.4094 clag sys-mac 44:38:39:FF:40:90
cumulus@switch:~$ net add bridge bridge ports server01,peerlink
cumulus@switch:~$ net add bridge stp treeprio 4096
cumulus@switch:~$ net add vlan 100 ip address 10.0.1.3/24
cumulus@switch:~$ net add vlan 100 ip address-virtual 44:38:39:FF:00:01 10.0.1.1/24
cumulus@switch:~$ net add vlan 200 ip address 10.0.2.3/24
cumulus@switch:~$ net add vlan 200 ip address-virtual 44:38:39:FF:00:02 10.0.2.1/24
cumulus@switch:~$ net add vlan 300 ip address 10.0.3.3/24
cumulus@switch:~$ net add vlan 300 ip address-virtual 44:38:39:FF:00:03 10.0.3.1/24
cumulus@switch:~$ net add vlan 400 ip address 10.0.4.3/24
cumulus@switch:~$ net add vlan 400 ip address-virtual 44:38:39:FF:00:04 10.0.4.1/24
cumulus@switch:~$ net pending
cumulus@switch:~$ net commit
```

These commands create the following configuration in `/etc/network/interfaces`:

```
auto eth0
iface eth0
    address 192.168.0.22
 
auto bridge
iface bridge
    bridge-ports server01 peerlink
    bridge-vids 100 200 300 400
    bridge-vlan-aware yes
    mstpctl-treeprio 4096
 
auto server01
iface server01
    alias LACP etherchannel to uplink on server01
    bond-slaves swp1 swp2
    clag-id 1
    mtu 9216
 
auto peerlink
iface peerlink
    bond-slaves swp49 swp50
 
auto peerlink.4094
iface peerlink.4094
    address 169.254.255.1/30
    clagd-backup-ip 192.168.0.22
    clagd-peer-ip 169.254.255.2
    clagd-sys-mac 44:38:39:FF:40:90
 
auto vlan100
iface vlan100
    address 10.0.1.3/24
    address-virtual 44:38:39:FF:00:01 10.0.1.1/24
    vlan-id 100
    vlan-raw-device bridge
 
auto vlan200
iface vlan200
    address 10.0.2.3/24
    address-virtual 44:38:39:FF:00:02 10.0.2.1/24
    vlan-id 200
    vlan-raw-device bridge
 
auto vlan300
iface vlan300
    address 10.0.3.3/24
    address-virtual 44:38:39:FF:00:03 10.0.3.1/24
    vlan-id 300
    vlan-raw-device bridge
 
auto vlan400
iface vlan400
    address 10.0.4.3/24
    address-virtual 44:38:39:FF:00:04 10.0.4.1/24
    vlan-id 400
    vlan-raw-device bridge
```

### server01 Configuration

Create a configuration like the following on an Ubuntu host:

```
auto eth0
iface eth0 inet dhcp
 
auto eth1
iface eth1 inet manual
    bond-master uplink
 
auto eth2
iface eth2 inet manual
    bond-master uplink
 
auto uplink
iface uplink inet static
    bond-slaves eth1 eth2
    bond-mode 802.3ad
    bond-miimon 100
    bond-lacp-rate 1
    bond-min-links 1
    bond-xmit-hash-policy layer3+4
    address 172.16.1.101
    netmask 255.255.255.0
    post-up ip route add 172.16.0.0/16 via 172.16.1.1
    post-up ip route add 10.0.0.0/8 via 172.16.1.1
 
auto uplink:200
iface uplink:200 inet static
    address 10.0.2.101
 
auto uplink:300
iface uplink:300 inet static
    address 10.0.3.101
 
auto uplink:400
iface uplink:400 inet static
    address 10.0.4.101
 
# modprobe bonding
```

### server02 Configuration

Create a configuration like the following on an Ubuntu host:

```
auto eth0
iface eth0 inet dhcp
 
auto eth1
iface eth1 inet manual
    bond-master uplink
 
auto eth2
iface eth2 inet manual
    bond-master uplink
 
auto uplink
iface uplink inet static
    bond-slaves eth1 eth2
    bond-mode 802.3ad
    bond-miimon 100
    bond-lacp-rate 1
    bond-min-links 1
    bond-xmit-hash-policy layer3+4
    address 172.16.1.101
    netmask 255.255.255.0
    post-up ip route add 172.16.0.0/16 via 172.16.1.1
    post-up ip route add 10.0.0.0/8 via 172.16.1.1
 
auto uplink:200
iface uplink:200 inet static
    address 10.0.2.101
 
auto uplink:300
iface uplink:300 inet static
    address 10.0.3.101
 
auto uplink:400
iface uplink:400 inet static
    address 10.0.4.101
 
# modprobe bonding
```

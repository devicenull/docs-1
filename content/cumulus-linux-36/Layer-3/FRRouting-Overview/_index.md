---
title: FRRouting Overview
author: Cumulus Networks
weight: 171
pageID: 8362387
---
Cumulus Linux uses FRRouting to provide the routing protocols for
dynamic routing. FRRouting provides many routing protocols, of which
Cumulus Linux supports the following:

  - Open Shortest Path First
    ([v2](/cumulus-linux-36/Layer-3/Open-Shortest-Path-First-OSPF-Protocol/)
    and
    [v3](/cumulus-linux-36/Layer-3/Open-Shortest-Path-First-v3-OSPFv3-Protocol/)
  - [Border Gateway Protocol](/cumulus-linux-36/Layer-3/Border-Gateway-Protocol-BGP/)

## Architecture

{{< img src = "/images/cumulus-linux/frrouting-overview-daemons.png" >}}

As shown in the figure above, the FRRouting suite consists of various
protocol-specific daemons and a protocol-independent daemon called
`zebra`. Each of the protocol-specific daemons are responsible for
running the relevant protocol and building the routing table based on
the information exchanged.

It is not uncommon to have more than one protocol daemon running at the
same time. For example, at the edge of an enterprise, protocols internal
to an enterprise (called IGP for Interior Gateway Protocol) such as
[OSPF](/cumulus-linux-36/Layer-3/Open-Shortest-Path-First-OSPF-Protocol/)
or RIP run alongside the protocols that connect an enterprise to the
rest of the world (called EGP or Exterior Gateway Protocol) such as
[BGP](/cumulus-linux-36/Layer-3/Border-Gateway-Protocol-BGP).

## About zebra

`zebra` is the daemon that resolves the routes provided by multiple
protocols (including static routes specified by the user) and programs
these routes in the Linux kernel via `netlink` (in Linux). `zebra` does
more than this, of course. The 
[FRRouting documentation](https://frrouting.org/user-guide/zebra.html) defines

`zebra` as the IP routing manager for FRRouting that "provides kernel
routing table updates, interface lookups, and redistribution of routes
between different routing protocols."

## Related Information

  - [frrouting.org](https://frrouting.org)
  - [GitHub](https://github.com/FRRouting/frr)

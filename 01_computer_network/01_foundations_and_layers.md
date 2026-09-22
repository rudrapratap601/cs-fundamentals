# 1. Foundations and Layers

[Index](README.md) · [Next](02_addressing_and_routing.md)

## Protocols and network types

A **protocol** defines message formats, ordering, and actions on communication events. A LAN spans a limited area; a WAN connects larger geographic regions. The Internet interconnects independently operated networks using IP.

Client/server describes roles: a client requests a service, and a server provides it. Peer-to-peer participants can both consume and provide resources. Unicast targets one destination; broadcast targets a broadcast domain; multicast targets a receiver group.

## OSI versus TCP/IP — core

OSI is a seven-layer reference model. The Internet stack is commonly taught with four layers; a five-layer model separates physical from link.

| OSI layer | Responsibility | Examples | Internet model |
|---|---|---|---|
| 7 Application | Application messages | HTTP, DNS, SMTP | Application |
| 6 Presentation | Representation and encoding | Serialization, character encoding | Application |
| 5 Session | Dialog/session coordination | Session management concepts | Application |
| 4 Transport | Process-to-process communication | TCP, UDP | Transport |
| 3 Network | Addressing/routing across networks | IP, ICMP | Internet |
| 2 Data link | Delivery on a local link | Ethernet, Wi-Fi | Link |
| 1 Physical | Signals | Copper, fiber, radio | Link |

Real protocols do not always fit neatly into OSI. TLS is commonly discussed between application protocols and transport; avoid treating OSI as an exact implementation blueprint.

### Encapsulation

```text
Application message → TCP segment/UDP datagram → IP packet → link frame → signals
```

The receiver removes headers. A router removes the incoming frame, examines the IP packet, and creates a frame for the next link. Link addresses normally change across routed hops. IP endpoints normally remain unchanged unless translation such as NAT occurs; TTL/hop limit changes along the path.

## Devices and switching

| Device | Typical behavior |
|---|---|
| Hub | Repeats signals without a learned destination table |
| Layer-2 switch | Forwards frames using destination MAC addresses |
| Router | Forwards IP packets using destination prefixes |
| Access point | Connects wireless clients, often bridging link traffic |
| Firewall | Allows/rejects traffic using policy, headers, and possibly connection/application state |

A switch learns **source MAC → ingress port** mappings. It forwards based on the destination MAC. Unknown unicast and broadcast frames are generally flooded within the relevant VLAN, excluding the ingress port. Routers normally separate broadcast domains; some physical devices perform both switching and routing.

**Circuit switching** reserves resources along a path. **Packet switching** shares capacity between packets, fitting bursty traffic but introducing queueing and possible loss.

## Performance — core

- **Bandwidth/capacity:** maximum data rate, in bits per second.
- **Throughput:** achieved delivery rate.
- **Goodput:** useful application payload rate, excluding overhead/retransmissions.
- **Latency:** delivery delay; **RTT** is round-trip time.
- **Jitter:** variation in delay.

```text
Transmission delay = packet bits / link bits per second
Propagation delay = distance / signal propagation speed
Nodal delay = processing + queueing + transmission + propagation
```

**Example:** Transmitting 1,500 bytes at 10 Mbps takes `1500 × 8 / 10,000,000 = 1.2 ms`. Propagating across 1,000 km at an assumed `2 × 10^8 m/s` takes `5 ms`. Ignoring queueing and processing, the last bit arrives after `6.2 ms` on this one link.

**Follow-up: bandwidth-delay product.** At 100 Mbps and 40 ms RTT, approximately `100,000,000 × 0.04 = 4,000,000 bits = 500,000 bytes` must be in flight to fill the path under simplified steady-state assumptions. A small transport window can restrict throughput.

## Interview answers and traps

**Does higher bandwidth always mean lower latency?** No. It lowers transmission time for a given packet but does not remove propagation distance or necessarily fix queueing.

**Why layers?** They separate responsibilities and support independent implementations. Costs include headers, processing, and imperfect abstraction boundaries.

**Switch versus router?** A typical Layer-2 switch moves frames within a LAN using MAC addresses; a router moves packets between networks using IP prefixes.

**Trap:** Convert bytes to bits before dividing by a link rate expressed in bps.

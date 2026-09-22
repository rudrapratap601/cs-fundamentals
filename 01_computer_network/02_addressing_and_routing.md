# 2. Addressing and Routing

[Index](README.md) · [Previous](01_foundations_and_layers.md) · [Next](03_transport_protocols.md)

## Address types

| Identifier | Purpose |
|---|---|
| MAC | Link-layer delivery on a local network |
| IP | Address an interface and route across networks |
| Port | Identify an application transport endpoint within a host/protocol |

Neither MAC nor IP is a permanent identity guarantee; addresses can change, and MAC addresses may be randomized.

## IPv4 and CIDR — core

IPv4 has 32 bits. `/n` means the first `n` bits are the network prefix; the remaining `32 − n` bits determine the block size.

```text
Block size = 2^(32 − prefix length)
Network address = IP address bitwise AND subnet mask
```

For conventional subnets through `/30`, usable host addresses usually equal `2^h − 2`, excluding network and broadcast addresses. `/31` point-to-point links and `/32` host routes are exceptions.

**Worked example: `192.168.10.77/26`.**

- Mask: `255.255.255.192`; six host bits; blocks of 64.
- Last-octet blocks: 0–63, 64–127, 128–191, 192–255.
- Network: `192.168.10.64`; broadcast: `192.168.10.127`.
- Usable: `192.168.10.65–192.168.10.126`, totaling 62 hosts.

For 50 hosts, five host bits give only 30 conventional usable addresses; six give 62, so `/26` fits.

| Range | Meaning |
|---|---|
| `10.0.0.0/8` | Private IPv4 |
| `172.16.0.0/12` | Private IPv4; not all `172.*` addresses |
| `192.168.0.0/16` | Private IPv4 |
| `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | IPv4 link-local |
| `0.0.0.0/0` | Default route prefix |

IPv6 has 128-bit addresses. It uses Neighbor Discovery rather than ARP and has multicast but no broadcast. IPv6 is not automatically encrypted or always faster.

## Local delivery and ARP

With a typical connected subnet and default route, a host determines whether a destination is on-link. For on-link delivery it resolves the destination's link address; otherwise it resolves the next-hop router's link address.

**ARP** maps an IPv4 next-hop address to a MAC address on the local network. A request is broadcast and replies are cached.

**Example:** A laptop sends to a remote server. The IP destination is the server, but the Ethernet destination is the gateway's MAC. The laptop does not ARP across the Internet for the server. ARP does not authenticate claims, making spoofing possible.

## DHCP, DNS, and NAT

**DHCP** supplies configuration such as an address lease, subnet mask, gateway, and DNS servers. A typical initial IPv4 exchange is **Discover → Offer → Request → Acknowledge**. Initial messages often use broadcasts; relays support servers on other subnets. Renewal need not repeat the full initial broadcast sequence.

**DNS** resolves names to records. A client usually asks a recursive resolver. Without cached information, that resolver may consult root, TLD, and authoritative servers. Root servers delegate; they do not store every host's IP address.

| Record | Meaning |
|---|---|
| A / AAAA | IPv4 / IPv6 address |
| CNAME | Alias to another name |
| MX | Mail exchanger |
| NS | Authoritative name server |
| TXT | Text used for multiple purposes |

TTLs guide caching. One name can return multiple addresses. Traditional DNS uses UDP or TCP port 53; encrypted transports also exist. DNS does not always use UDP and generally does not return a full URL.

**NAT** rewrites network addresses. **PAT** uses port mappings to let multiple private endpoints share a public IPv4 address:

```text
192.168.1.10:51000 → server:443
public-address:62001 → server:443  (after translation)
```

Return traffic is mapped back. NAT conserves public IPv4 addresses but complicates inbound connections and end-to-end communication. It is distinct from firewall policy.

## Routing — core and follow-up

Forwarding uses the **longest matching prefix**:

```text
10.0.0.0/8  → A
10.1.0.0/16 → B
0.0.0.0/0  → C
10.1.2.3 matches all three; choose /16 → B.
```

Routing builds/selects routes; forwarding applies them to packets. Static routes are configured manually; dynamic protocols exchange reachability. OSPF is a link-state interior protocol. BGP exchanges inter-domain reachability and applies policy; it does not simply minimize physical distance.

IPv4 TTL and IPv6 hop limit bound forwarding loops. ICMP carries diagnostic/control messages. Ping commonly uses ICMP echo; traceroute uses increasing TTL/hop-limit probes. Filtering/rate limits can make missing responses inconclusive.

## Interview answers

**ARP versus DNS?** Local IPv4-to-link-address resolution versus name-to-record resolution.

**No default gateway?** On-link communication can still work; off-link destinations require an appropriate route and next hop.

**Do DNS updates appear immediately?** Cached answers may persist for their applicable lifetime; clients and resolvers can differ.

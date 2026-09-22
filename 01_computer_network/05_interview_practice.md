# 5. CN Interview Practice

[Index](README.md) · [Previous](04_web_protocols_and_security.md)

## Rapid recall

| Question | Answer checkpoint |
|---|---|
| What changes at a routed hop? | Link frame/addresses and TTL/hop limit; IP endpoints normally persist without translation |
| Why is TCP reliable? | Sequence numbers, ACKs, retransmission, ordering, duplicate handling |
| Flow versus congestion control? | Receiver capacity versus network capacity |
| Can a UDP application be reliable? | Yes; higher-level protocols can implement reliability |
| ARP, DNS, DHCP? | Local IPv4-to-MAC mapping, name records, configuration |
| Why TIME_WAIT? | Old segment expiration and final-ACK retransmission |
| Why ports? | Distinguish transport endpoints on a host |
| Does stateless HTTP prohibit login? | No; applications add sessions/tokens |

## Problems with answers

### Subnetting

**Question:** Network, broadcast, and usable range for `10.20.30.140/27`?

**Answer:** Five host bits give blocks of 32. Network `.128`; broadcast `.159`; usable `.129–.158`, 30 addresses. Mask `255.255.255.224`.

### Delay

**Question:** Transmission time for 1,000 bytes at 20 Mbps?

**Answer:** `8000 / 20,000,000 = 0.0004 seconds = 0.4 ms`. This excludes propagation, queueing, processing, and further links.

### TCP sequence numbers

**Question:** Bytes 2000–2499 arrive in order; next cumulative ACK?

**Answer:** 2500, assuming no earlier gap. TCP counts bytes, not packets.

### Name lookup failure

**Question:** A service works by IP but not by name. What do you inspect?

**Answer:** Resolved records, resolver reachability, caches, and local overrides. For HTTPS, raw-IP access can change hostname verification and virtual-host routing, so it is not always an equivalent application test.

### Slow website

**Question:** Why might a website be slow despite high bandwidth?

**Answer:** Separate DNS, connection/TLS setup, server response, transfer, and rendering time. Investigate RTT, loss, backend/database delays, request count, and caches.

### Timed-out order

**Question:** Is it safe to assume an order failed after a timeout?

**Answer:** No; it may have committed before the response was lost. Use operation status and an appropriately designed idempotent retry mechanism.

## Explain aloud

1. Laptop to remote server: include route, gateway ARP, different frame/IP destinations, and forwarding.
2. HTTP/2 blocking: explain multiple streams sharing one ordered TCP stream.
3. HTTPS verification: include certificate chain, hostname, private-key proof, and traffic keys.
4. Proxy versus router versus resolver: distinguish application mediation, packet forwarding, and naming.

## Mistakes to eliminate

- “TCP proves that a request was processed.”
- “DNS always uses UDP.”
- “The laptop ARPs for the remote Internet server.”
- “UDP is always faster.”
- “`no-cache` means do not store.”
- “`2^h − 2` has no subnetting exceptions.”

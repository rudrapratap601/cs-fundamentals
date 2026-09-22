# 3. Transport Protocols

[Index](README.md) · [Previous](02_addressing_and_routing.md) · [Next](04_web_protocols_and_security.md)

## Ports and connections

TCP and UDP have separate port namespaces. A TCP connection is commonly identified by source IP/port and destination IP/port, with TCP understood. Many clients can use one server port because their connection tuples differ.

Common conventions: SSH TCP 22; DNS UDP/TCP 53; HTTP TCP 80; HTTPS TCP 443, with HTTP/3 commonly on UDP 443; SMTP relay TCP 25. A port is a convention, not proof of an application's identity.

## TCP versus UDP — core

| Property | TCP | UDP |
|---|---|---|
| Model | Ordered byte stream | Datagrams |
| Setup | Connection establishment | No transport handshake |
| Reliability | Retransmission, ordering, duplicate handling | No built-in delivery/ordering guarantee |
| Message boundaries | Not preserved | Preserved for delivered datagrams |
| Control | Flow and congestion control | Higher protocol/application must provide appropriate control |
| Typical fit | Reliable streams | Low-overhead messages, real-time traffic, custom transports |

TCP connections can fail; TCP does not prove remote application processing. UDP is not always faster. QUIC builds a reliable secure transport over UDP.

## Three-way handshake

```text
Client                         Server
SYN seq=x              →
                       ← SYN+ACK seq=y, ack=x+1
ACK ack=y+1            →
```

This synchronizes sequence numbers and confirms bidirectional communication. SYN consumes one sequence number. With only two messages, the server lacks confirmation that its SYN reached the client.

**Follow-up:** SYN floods try to consume resources with incomplete handshakes. Defenses can include SYN cookies and traffic controls.

## Reliability

- Sequence numbers identify byte positions.
- Cumulative ACKs normally identify the next expected byte.
- Timeouts and other loss evidence trigger retransmissions.
- Receivers buffer out-of-order data and deliver an ordered stream.
- Checksums detect corruption but are not cryptographic authentication.

**Example:** A segment starting at 1000 with 500 payload bytes can be acknowledged with 1500 if received in order. If bytes 1500–1999 are missing, later data cannot advance the cumulative ACK past 1500. Selective acknowledgments can report later received ranges.

Two `send()` calls need not produce two `recv()` calls. Application messages need framing, such as lengths or delimiters.

## Flow versus congestion control

| Control | Protects | Main concept |
|---|---|---|
| Flow | Receiver buffers | Advertised receive window, `rwnd` |
| Congestion | Network capacity | Congestion window, `cwnd` |

Outstanding data is roughly bounded by `min(rwnd, cwnd)`. Sending also depends on existing in-flight data and other constraints.

Classic TCP algorithms use slow start, congestion avoidance, and reactions to loss. Slow start grows the window rapidly; congestion avoidance grows more cautiously. Specific algorithms differ—do not assert identical window changes for every TCP implementation.

**Head-of-line blocking:** Missing earlier bytes delay delivery of later bytes in an ordered TCP stream.

## Closing a connection

TCP is full duplex. Each direction can close using FIN/ACK, often illustrated with four messages; messages may be combined. FIN consumes a sequence number. RST aborts a connection.

In the usual active-close sequence, the endpoint sending the final ACK enters **TIME_WAIT** so old segments expire and the last ACK can be retransmitted if needed. It is not necessarily the server; simultaneous close can change the usual roles.

## Interview answers

**Why use UDP for media?** Late data may be less valuable than missing data. Applications can choose their own buffering, loss recovery, and adaptation, while still behaving responsibly under congestion.

**Does a TCP ACK prove an order was saved?** No. It acknowledges transport receipt, not a database commit.

**What if an ACK is lost?** A later cumulative ACK may cover it; otherwise data may be retransmitted and recognized as duplicate.

**Does connection-oriented mean a reserved circuit?** No. TCP maintains endpoint state over packet-switched networks.

---
{"dg-publish":true,"dg-permalink":"tcp-vs-udp","permalink":"/tcp-vs-udp/","created":"2024-06-17T23:34:21.276+08:00","updated":"2024-06-18T00:07:07.227+08:00"}
---

#Network #HTTP #TCP #UDP
### General Overview
- **TCP (Transmission Control Protocol)**
    - Ensures reliable data transfer with error checking and retransmission.
    - Guarantees ordered delivery of packets.
    - Establishes a connection through a three-way handshake before data transfer.
    - Commonly used for applications where reliability is crucial (e.g., web browsing, email).
- **UDP (User Datagram Protocol)**
    - Provides a faster but less reliable data transfer.
    - No connection setup, and minimal error checking.
    - Does not guarantee packet order or delivery.
    - Suitable for applications where speed is essential and occasional data loss is acceptable (e.g., video streaming, online gaming).
To summarize in a table,

| Factor              | TCP                                                                   | UDP                                                                     |
| ------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Connection type     | Requires an established connection before transmitting data           | No connection is needed to start and end a data transfer                |
| Data sequence       | Can sequence data (send in a specific order)                          | Cannot sequence or arrange data                                         |
| Data retransmission | Can retransmit data if packets fail to arrive                         | No data retransmitting. Lost data can’t be retrieved                    |
| Delivery            | Delivery is guaranteed                                                | Delivery is not guaranteed                                              |
| Check for errors    | Thorough error-checking guarantees data arrives in its intended state | Minimal error-checking covers the basics but may not prevent all errors |
| Broadcasting        | Not supported                                                         | Supported                                                               |
| Speed               | Slow, but complete data delivery                                      | Fast, but at risk of incomplete data delivery                           |

#### how TCP works
- **Connection Establishment (Three-way Handshake)**
    1. **SYN**: The client sends a SYN (synchronize) packet to the server, initiating the connection and specifying the initial sequence number.
    2. **SYN-ACK**: The server responds with a SYN-ACK packet, acknowledging the client's SYN and providing its initial sequence number.
    3. **ACK**: The client sends an ACK (acknowledgment) packet back to the server, completing the handshake and establishing the connection.
- **Data Transfer**
    - Once the connection is established, data can be sent between the client and server. TCP ensures data integrity through sequence numbers and acknowledgments.
- **Connection Termination**
    - To close the connection, a four-step process (FIN, FIN-ACK, ACK) is typically used, ensuring that both sides properly terminate the connection.
### Additional Insights
- **Flow Control**: TCP uses flow control mechanisms like the sliding window protocol to manage the rate of data transmission between sender and receiver, preventing congestion.
- **Congestion Control**: TCP also incorporates congestion control algorithms (e.g., slow start, congestion avoidance) to avoid network congestion.
- **UDP Use Cases**: While UDP lacks the reliability of TCP, its low latency makes it ideal for real-time applications such as live broadcasts, and online gaming.
- **Security Considerations**: 
	- TCP is more susceptible to certain types of attacks, such as SYN flooding, due to its connection-oriented nature.
	- UDP can be more challenging to secure because it is connectionless and often used for broadcasting.
### Modern Usage of TCP and UDP

#### HTTP (HyperText Transfer Protocol)
- **HTTP**: Uses TCP as its underlying transport protocol.
    - Relies on the reliable, ordered, and error-checked delivery of TCP.
    - HTTP/1.1 introduced persistent connections, reducing the overhead of establishing a new TCP connection for each request.
#### HTTP/2
- **HTTP/2**: Also uses TCP but introduces significant improvements over HTTP/1.x.
    - Multiplexing: Allows multiple requests and responses to be sent simultaneously over a single TCP connection, reducing latency.
    - Header Compression: Compresses headers to reduce overhead and improve performance.
    - Server Push: Enables servers to send resources to clients proactively, reducing load times.
    - Binary Protocol: Uses a binary protocol instead of a text-based protocol, making parsing more efficient.
#### HTTP/3
- **HTTP/3**: Uses UDP instead of TCP, leveraging the QUIC (Quick UDP Internet Connections) protocol.
    - QUIC: Developed by Google, QUIC aims to combine the reliability of TCP with the speed of UDP.
    - Connection Establishment: Faster connection establishment compared to TCP's three-way handshake, improving latency.
    - Multiplexing: Like HTTP/2, HTTP/3 supports multiplexing without the head-of-line blocking issue present in TCP.
    - Improved Congestion Control: QUIC includes advanced congestion control mechanisms.
    - Built-in Encryption: QUIC includes encryption by default, enhancing security.

#### Summary

| Protocol | Transport Layer | Key Features                                                                                                                   |
| -------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| HTTP     | TCP             | Reliable, ordered, error-checked delivery. Persistent connections in HTTP/1.1.                                                 |
| HTTP/2   | TCP             | Multiplexing, header compression, server push, binary protocol.                                                                |
| HTTP/3   | UDP (QUIC)      | Faster connection establishment, multiplexing without head-of-line blocking, improved congestion control, built-in encryption. |

#### Reference
- [TCP vs UDP: Differences between the protocols](https://www.avast.com/c-tcp-vs-udp-difference)

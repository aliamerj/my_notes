# Network Communication in Systems Programming
Moving from establishing network connections to actually sending and receiving data over the network. 
This involves using socket functions for sending (via send) and receiving (via recv). 

## General Overview of Network Communication
Network Communication involves data exchange between systems using protocols like TCP (Transmission Control Protocol) and UDP (User Datagram Protocol). These are built on top of the IP (Internet Protocol) layer.
  1. TCP is connection-oriented, reliable, and ensures data arrives in order, but it has overhead.
  2. UDP is connectionless and does not guarantee delivery or order, but it is faster and used in time-sensitive applications (e.g., streaming, gaming).
**Sockets are the basic building blocks of network communication in many operating systems. They allow sending and receiving data over a network using these protocols.**


## Sending Data: send()
The function send() is used to send data over an established connection. It takes the following parameters:
1. sockfd: The socket file descriptor.
2. msg: Pointer to the data to send.
3. length: Length of the data in bytes.
4. flags: Flags to modify the behavior of the call (use 0 for simple use cases).

The function returns the number of bytes sent, or -1 if there is an error (in which case, `errno` can provide more information).
In large data transfers, you may need to send data in **chunks**, as send() might not send all bytes in one call. A helper function sendall() can repeatedly send data until everything is sent.

## Receiving Data: recv()
The recv() function is used to receive data over a socket. It takes similar parameters to send():
1. sockfd: The socket file descriptor.
2. buffer: The buffer to store the received data.
3. length: The maximum length of data to receive.
4. flags: Flags (set to 0 for simple cases).

The function returns the number of bytes received, 0 if the connection is closed, or -1 if an error occurs.

## Datagram Communication
For connectionless communication, like using UDP, the functions sendto() and recvfrom() are used. These allow you to specify the address of the receiver or sender in each call, since no connection is maintained.
cURL for Network Requests

**When communicating with web services or URLs, the cURL library simplifies the process. It offers a higher-level API for network communication, making it easier to send requests (e.g., GET, POST) and handle responses without manually managing sockets.**

## Zig Example: Sending and Receiving Data
Sending Data with Zig

The Zig equivalent of send() is achieved by using the `std.os.send` function. Here’s a simple example of sending data using Zig:
```zig
const std = @import("std");

pub fn sendData(sockfd: std.os.fd_t) !void {
    const msg = "Hello, world!";
    const len = msg.len;

    var bytes_sent = try std.os.send(sockfd, msg[0..], 0);

    while (bytes_sent < len) {
        const remaining = msg[bytes_sent..];
        bytes_sent += try std.os.send(sockfd, remaining, 0);
    }
}
```
In this Zig function:
1. sockfd: The socket file descriptor.
2. We use std.os.send to send the message in chunks, ensuring that all the data is sent.

## Receiving Data with Zig
For receiving data, Zig provides `std.os.recv`. Here’s an example:
```zig
const std = @import("std");

pub fn receiveData(sockfd: std.os.fd_t) ![]u8 {
    var buffer: [1024]u8 = undefined;
    const bytes_received = try std.os.recv(sockfd, &buffer, 0);

    return buffer[0..bytes_received];
}
```
In this function:
1. We receive data into a buffer of 1024 bytes.
2. The function returns the slice of the buffer containing the actual received data.

# Datagram Communication with Zig (UDP)

For datagram communication, Zig also provides the necessary functionality, such as `std.os.sendto` and `std.os.recvfrom`. This allows you to send and receive data over UDP without establishing a persistent connection.
Summary of Key Concepts:
1. Send: Use send() in C or std.os.send in Zig to transmit data over a connection. Be mindful of network issues and partial sends.
2. Receive: Use recv() or std.os.recv in Zig to receive data, and handle cases where the other side closes the connection.
3. Datagrams: For connectionless communication, use sendto() and recvfrom() in C or equivalent functions in Zig.
4. cURL: For higher-level network communication, use a library like cURL to simplify sending and receiving HTTP requests without manually managing sockets.

## Network Communication in Linux
- Linux provides a powerful set of tools for network programming through sockets. There are several types of sockets, including:
    1. Stream Sockets (SOCK_STREAM): For TCP-based communication.
    2. Datagram Sockets (SOCK_DGRAM): For UDP-based communication.

- System calls involved in socket programming:
    - socket(): Creates a socket.
    - bind(): Binds the socket to a specific IP and port.
    - listen(): Listens for incoming connections (for TCP).
    - accept(): Accepts an incoming connection.
    - connect(): Initiates a connection (for TCP).
    - send(), recv(): Sends and receives data over the socket.
    - sendto(), recvfrom(): For UDP communication, where connections aren’t established.
    - close(): Closes the socket.

**Non-blocking sockets and epoll: For high-performance applications, non-blocking sockets allow operations to continue without waiting for data to arrive. Epoll is used to monitor multiple file descriptors, including sockets, to determine which ones are ready for I/O, enabling efficient event-driven programming.**

## Network Communication in Zig
Zig provides low-level control over sockets, similar to C, but with better memory safety and performance guarantees.
  Key Zig networking types:
  - std.net.Address: Represents an IP address (IPv4 or IPv6).
  - std.net.StreamServer: A TCP server type.
  - std.net.StreamClient: A TCP client type.
  - std.net.datagram: Used for UDP communication.

## Example of TCP server in Zig:
```zig
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    var server = try std.net.StreamServer.init(.{
        .address = try std.net.Address.parseIp4("0.0.0.0", 1234),
    });
    defer server.deinit();

    try server.listen();

    while (true) {
        const conn = try server.accept(allocator);
        defer conn.close();

        const data = try conn.reader().readAllAlloc(allocator, std.math.maxInt(usize));
        defer allocator.free(data);
        
        try conn.writer().writeAll(data);
    }
}
```
- This example shows a simple TCP echo server in Zig. It listens for incoming connections, reads data, and then echoes it back.

- Non-blocking mode can also be implemented in Zig using system calls like fcntl() or by using Zig's own async/await pattern.

## Network Communication Performance
Performance is critical in network communication, especially in systems where high throughput or low latency is needed (e.g., web servers, databases, real-time systems).
- Epoll (Linux) allows applications to efficiently handle multiple connections. Instead of constantly polling for data (which consumes CPU cycles), epoll waits for events on multiple file descriptors (including sockets).
- Zig's event loop: Zig provides async/await functionality, allowing non-blocking I/O without explicitly managing epoll or similar APIs. Zig's approach to concurrency with async/await can make it easier to manage many network connections.

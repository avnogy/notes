---
layout: post
title: Scapy Raw Socket Loopback in Linux
categories: networking
author: avnogy
tags:
  - networking
  - scapy
  - rawsocket
  - linux
---

While attempting to simulate a TCP 3-way handshake using Scapy, I encountered an issue where I did not get get a response from `127.0.0.1` on the loopback interface.
Running my code on a different interface using two different IP addresses worked fine, but this was interesting.
## The Initial try
I used Scapy for manually creating the handshake and a Python socket as the server to see we can communicate with a standard application.

My manual code:
```python
from scapy.all import *

ip_layer = IP(src="127.0.0.1", dst="127.0.0.1")

syn_packet = ip_layer / TCP(sport=1234, dport=8080, flags="S")
response = sr1(syn_packet, timeout=1, verbose=0)

if not response:
    print("No response received from the server.")
    quit()
ack_num = response.seq + 1

ack_packet = ip_layer / TCP(sport=1234, dport=8080, flags="A", seq=1234, ack=ack_num)
send(ack_packet, verbose=0)
```

My built-in socket code:
```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("127.0.0.1", 8080))
server_socket.listen(1)
print("Server listening on port 8080...")

client_socket, address = server_socket.accept()
```

After running the sockets code and the manual code, we can see in `tcpdump` that we sent a SYN packet, but we got no response.

![](/notes/assets/code/scapyrawsocket/Pasted%20image%2020250301200358.png)
> We would expect to get either a SYN+ACK segment from our sockets code, or an RST segment if something went wrong, but getting nothing is weird.

## Troubleshooting Scapy with Loopback Interface

After looking around and making sure the problem was related specifically to localhost and to my implementation using Scapy, I found [a section in the Scapy troubleshooting page](https://scapy.readthedocs.io/en/latest/troubleshooting.html#i-can-t-ping-127-0-0-1-or-1-scapy-does-not-work-with-127-0-0-1-or-1-on-the-loopback-interface) that explained what was going on. 

The issue arises because in Linux, the lo interface is a special interface where packets are not fully assembled and disassembled. Instead, the kernel routes the packet to its destination while it is still stored in an internal structure. This means that what we saw with `tcpdump` is not the actual packet transmission but rather a simulated representation.

Practically, this also means that the kernel doesn't create IP headers for packets routed using this interface, and therefore it doesn't deal with any non L3 packets. Because we created L2 packets, they weren't parsed, and we got no response.

Here is an image I found in a random website in Chinese[^1] illustrating this concept:
![](/notes/assets/code/scapyrawsocket/Pasted%20image%2020250301203330.png)

## The solution

We need to make sure we conform to the lo interface's way of doing things, meaning we need to use a different kind of raw-socket. By default, Scapy uses an L2 raw-socket (PF_PACKET), We need to use an L3 raw-socket (PF_INET) instead.

We can do that by running:
```python
conf.L3socket = L3RawSocket
```

By making this adjustment, we can successfully use Scapy with the loopback interface on Linux systems.
![](/notes/assets/code/scapyrawsocket/Pasted%20image%2020250301200310.png)


[^1]: <https://www.cnblogs.com/jiujuan/p/9388517.html>

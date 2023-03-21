# Generating IPv6 Extension Headers

This guide gives a basic overview of Scapy and how to create simply IPv6 packets with extension headers.

## Goals

- Learn basic scapy commands
- Send and receive IPv6 packets
- Create IPv6 packets with different extension headers

## Prerequisites

- Have Python 3.\* installed

```
sudo apt update
sudo apt install python3
```

- Install the scapy python package with commands as shown below.

```
python3 -m venv env

# On Linux
source env/bin/activate

# On Windows
env\Scripts\activate

pip install scapy
```

- Install the linux scapy package as show below.

```
sudo apt-get update
sudo apt-get install scapy
```

- Install Wireshark

```
sudo apt update
sudo apt install wireshark
```

## Task 1: Try out basic scapy commands

### Create and send a basic IPv4 packet

Run the following commands.

```
sudo scapy
>>> packet = IP()
>>> packet.src = "192.168.0.1"
>>> packet.dst = "192.168.0.2"
>>> packet.show()
```

Expected output:

```
###[ IP ]###
  version   = 4
  ihl       = None
  tos       = 0x0
  len       = None
  id        = 1
  flags     =
  frag      = 0
  ttl       = 64
  proto     = ip
  chksum    = None
  src       = 192.168.0.1
  dst       = 192.168.0.2
  \options   \
```

Send the created packet on the network

```
>>> send(packet)
```

These packets and be monitored on wireshark as shown in the demo.

Try modifying other fields of the packets like time to live (TTL) and observe the changes on wireshark.

Additionally the above commands can be simplified to a one liner!

```
>>> send(IP(src="192.168.0.1", dst="192.168.0.2"))
```

### Create and send a basic IPv6 packet

This is very similar to creating an IPv4 packets. Run the following command.

```
>>> ipv6_packet = IPv6(src="fe80::1111", dst="fe80::2222")
>>> ipv6_packet.show()
```

Expected output:

```
###[ IPv6 ]###
  version= 6
  tc= 0
  fl= 0
  plen= None
  nh= No Next Header
  hlim= 64
  src= fe80::1111
  dst= fe80::2222
```

Now send the packet and and observe it on wireshark

```
>>> send(ipv6_packet)
```

### "Stack" protocols

You can “stack” protocols on top of each other, just like a real network stack would, by using the Slash-Operator
(/):

```
>>> ipv6_tcp = IPv6() / TCP()
>>> ipv6_tcp.show()
```

This creates a TCP-over-IPv6-Packet.

Expected output:

```
###[ IPv6 ]###
  version   = 6
  tc        = 0
  fl        = 0
  plen      = None
  nh        = TCP
  hlim      = 64
  src       = ::1
  dst       = ::1
###[ TCP ]###
     sport     = ftp_data
     dport     = http
     seq       = 0
     ack       = 0
     dataofs   = None
     reserved  = 0
     flags     = S
     window    = 8192
     chksum    = None
     urgptr    = 0
     options   = ''
```

## Task 2: Create IPv6 Packet with Extension Headers

### First let us look into some of the default IPv6 extension headers build into scapy

- Hop-By-Hop
- Destination
- Routing
- Fragment

These are all defined in detail in [RFC2460](https://www.rfc-editor.org/rfc/pdfrfc/rfc2460.txt.pdf).

Run the following commands to see how the extension headers classes is defined in scapy respectively.

```
>>> ls(IPv6ExtHdrHopByHop)
>>> ls(IPv6ExtHdrDestOpt)
>>> ls(IPv6ExtHdrRouting)
>>> ls(IPv6ExtHdrFragment)
```

### Create an IPv6-Jumbogram using IPv6 Extension Headers

A jumbogram is an internet-layer packet exceeding the standard maximum transmission unit (MTU) of the underlying network technology.

First create the base IPv6 packet similar to before.

```
>>> ipv6_base = IPv6()
>>> ipv6_base.src = 'fe80::1111'
>>> ipv6_base.dst = 'fe80::2222'
```

Now we create a Hop by Hop extension header and sent the Jumbograph-Option in it.

```
>>> extension = IPv6ExtHdrHopByHop()
>>> jumbo = Jumbo()
```

We choose a arbitarily large number 32 bit number to specify the payload length of the jumbogram.

```
>>> jumbo.jumboplen = 2**30
>>> extension.options = jumbo
```

Finally, we stack the headers to create the final packet.

```
>>> packet = ipv6_base / extension
>>> packet.show()
```

Expected Output:

```
###[ IPv6 ]###
  version   = 6
  tc        = 0
  fl        = 0
  plen      = None
  nh        = Hop-by-Hop Option Header
  hlim      = 64
  src       = fe80::1111
  dst       = fe80::2222
###[ IPv6 Extension Header - Hop-by-Hop Options Header ]###
     nh        = No Next Header
     len       = None
     autopad   = On
     \options   \
      |###[ Jumbo Payload ]###
      |  otype     = Jumbo Payload [11: discard+ICMP not mcast, 0: Don't change en-route]
      |  optlen    = 4
      |  jumboplen = 1073741824
```

Now the packet is ready to be sent.

```
>>> send(packet)
```

Congratulations! You have successfully created an IPv6 packet with extension headers!!

**Bonus Task:** Try to perform the above task in just one line!

# Resources

- [Official Scapy Documentation](https://scapy.readthedocs.io/en/latest/usage.html)
- [IPv6 Packet Creation With Scapy
  Documentation by Oliver Eggert](https://www.idsv6.de/Downloads/IPv6PacketCreationWithScapy.pdf)

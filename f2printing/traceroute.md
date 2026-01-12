# Traceroute

## Overview

Traceroute is a **fingerprinting** network diagnostic tool used to track the pathway that packets take from one computer to another across an IP network. It helps identify the route and measure transit delays of packets across the network. Traceroute works by sending a series of Internet Control Message Protocol (ICMP) Echo Request messages to the destination, with incrementally increasing Time-To-Live (TTL) values. Each router along the path decrements the TTL value by one before forwarding the packet. When the TTL reaches zero, the router sends back an ICMP Time Exceeded message to the sender, revealing its IP address. This process continues until the packet reaches its destination.

## Installation

Traceroute is typically pre-installed on most Unix-like operating systems, including Linux and macOS. On Windows, a similar tool called `tracert` is available by default. If not, simply use the package manager to install it.

## Usage

To use traceroute, simply run the following command:

```bash
traceroute [options] <destination>
```
    
For example, to trace the route to example.com, you would use:
```bash
$ traceroute example.com

traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  router.local (192.168.1.1)  2.011 ms  2.005 ms  1.973 ms
 2  isp-gateway.local (10.0.0.1)  5.014 ms  5.045 ms  5.089 ms
 3  * * * 
 4  core-router-2.isp.com (203.0.113.2)  15.204 ms  15.187 ms  15.171 ms
 5  edge-router.isp.com (198.51.100.1)  20.996 ms  21.010 ms  20.982 ms
 6  * * * 
 7  example.com (93.184.216.34)  28.123 ms  28.143 ms  28.130 ms

```

## Relevant Options

Disabling IP address and host name mapping is particularly useful when speed is of the essence. By bypassing the DNS resolution stage, which can slow down the process. Ideal when the primary goal is to discern the network path without the additional layer of DNS translation mechanics.

```bash
traceroute -n example.com
```

By specifying a wait time, it is manageable how long the traceroute command should wait for a response from each hop. In cases where network latency varies, or when testing a connection over volatile networks, adjusting this wait time allows for capturing data that respects network conditions and personal thresholds for acceptable wait times. The custom wait time is set in seconds.

```bash
traceroute -w 0.5 example.com
```

MTU is the largest size (in bytes) a packet can be for transmission without needing to be fragmented.
Understanding the Maximum Transmission Unit (MTU) size for the path to a destination might be useful in a variety of scenarios.

```bash
$ traceroute --mtu example.com

traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets, MTU discovery
1  192.168.1.1 (MTU=1500)  1.118 ms  1.117 ms  1.114 ms
2  10.0.0.1 (MTU=1400)  2.482 ms  2.480 ms  2.478 ms
3  192.0.2.1 (MTU=1300)  3.953 ms  3.951 ms  3.948 ms
```

In networks where UDP traffic is restricted or filtered, using ICMP or TCP for tracerouting can be interesting as they can often bypass such restrictions.

```bash
traceroute -I example.com            # Use ICMP ECHO for tracerouting
traceroute -T example.com            # Use TCP SYN for tracerouting
```
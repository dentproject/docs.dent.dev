---
title: Stateless Network Address Translation
grand_parent: Network Configuration
parent: Network Addressing and Filtering
nav_order: 4
layout: default
---

# Stateless Network Address Translation (NAT)

Stateless Network Address Translation (NAT), also known as static NAT, establishes a permanent mapping between a private IP address and a single public address.

To configure stateless NAT, the `tc iproute2` tool is utilized, along with the `tc-nat` action. This action is combined with the `flower` filter rule and `ingress` qdisc to enable static NAT entry offloading. Below is the command format:

```
tc ... flower ... action nat { ingress | egress } <OLD> <NEW>
```

Where:

- `<OLD>` represents the IP address to be translated.
- `<NEW>` represents the IP address to translate to.
- `ingress` translates destination addresses (performs DNAT).
- `egress` translates source addresses (performs SNAT).

To implement NAT one-to-one mapping, two rules are required: one for private-to-public direction and another for public-to-private direction.

### Example Configuration

Configure the private host (Assume it to be PC1):

```
ip 192.168.0.2/24 192.168.0.1
```

Configure the public host (Assume it to be PC2):

```
ip 91.245.77.2/24
```

Configure IP on the interfaces and set up a default gateway:

```
ip addr add dev swp23 192.168.0.1/24
ip addr add dev swp24 91.245.77.1/24
```

```
ip route add default via 91.245.77.1
```

Ensure that the interfaces are up. You can bring up the interfaces using the following commands:

```
ip link set dev swp23 up
ip link set dev swp24 up
```

![Stateless-NAT](../../Images/ImagesForNetworkConfiguration/NAT2.png)

This setup ensures PC1 (the private host) is on the private network, and PC2 (the public host) is on the public network, with the DENT VM managing the connection tracking and Stateless NAT between these networks.

Configure ACL rules for NAT offloading:

```
tc qdisc add dev swp23 clsact
```

NAT connection tracking on `swp23` for egress traffic:

```
tc filter add dev swp23 protocol ip egress flower ip_proto tcp src_ip 192.168.0.2 action nat egress 192.168.0.2 91.245.77.1
```

```
filter protocol ip pref 49152 flower chain 0
filter protocol ip pref 49152 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  src_ip 192.168.0.2
  not_in_hw
        action order 1:  nat egress 192.168.0.2/32 91.245.77.1 pass
         index 1 ref 1 bind 1
```

NAT connection tracking on `swp23` for ingress traffic:

```
tc filter add dev swp23 protocol ip ingress flower ip_proto tcp dst_ip 91.245.77.1 action nat ingress 91.245.77.1 192.168.0.2
```

```
filter protocol ip pref 49152 flower chain 0
filter protocol ip pref 49152 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  dst_ip 91.245.77.1
  in_hw in_hw_count 1
        action order 1:  nat ingress 91.245.77.1/32 192.168.0.2 pass
         index 2 ref 1 bind 1
        used_hw_stats delayed
```

To create only HW rule on `swp24`:

```
tc qdisc add dev swp24 clsact
```

```
tc filter add dev swp24 protocol ip ingress flower skip_sw ip_proto tcp src_ip 192.168.0.2 action nat egress 192.168.0.2 91.245.77.1
```

```
filter protocol ip pref 49152 flower chain 0
filter protocol ip pref 49152 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  src_ip 192.168.0.2
  skip_sw
  in_hw in_hw_count 1
        action order 1:  nat egress 192.168.0.2/32 91.245.77.1 pass
         index 3 ref 1 bind 1
        used_hw_stats delayed
```

**Verify Connectivity**:

From PC1 to DENT:

```
PC1> ping 192.168.0.1
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.056 ms
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=0.033 ms
64 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=0.047 ms
64 bytes from 192.168.0.1: icmp_seq=4 ttl=64 time=0.045 ms
```

From PC2 to DENT:

```
PC1> ping 91.245.77.1
PING 91.245.77.1 (91.245.77.1) 56(84) bytes of data.
64 bytes from 91.245.77.1: icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from 91.245.77.1: icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from 91.245.77.1: icmp_seq=3 ttl=64 time=0.047 ms
64 bytes from 91.245.77.1: icmp_seq=4 ttl=64 time=0.041 ms
```

From PC1 to PC2:

```
PC1> ping 91.245.77.2
PING 91.245.77.2 (91.245.77.2) 56(84) bytes of data.
64 bytes from 91.245.77.2: icmp_seq=1 ttl=64 time=0.046 ms
64 bytes from 91.245.77.2: icmp_seq=2 ttl=64 time=0.044 ms
64 bytes from 91.245.77.2: icmp_seq=3 ttl=64 time=0.040 ms
64 bytes from 91.245.77.2: icmp_seq=4 ttl=64 time=0.035 ms
```

Since all three pings are successful, it indicates that the network is configured correctly and connectivity is established.

## Private to Private Flow

To skip NAT for packets designated as a private subnet or hosts:

```
tc filter add dev swp24 protocol ip ingress flower skip_sw ip_proto tcp dst_ip 192.168.0.1/24 action pass
```

```
filter protocol ip pref 49152 flower chain 0
filter protocol ip pref 49152 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  dst_ip 192.168.0.1/24
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action pass
         random type none pass val 0
         index 5 ref 1 bind 1
        used_hw_stats delayed
```

**Note:** The last added rule has a higher priority, so the priority can be skipped in the rule.

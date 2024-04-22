---
title: Policing Data Path Traffic
grand_parent: Network Configuration
parent: Traffic Control and Policing
nav_order: 2
layout: default
---

# Policing Data Path Traffic

Configuring the policing of data path traffic is similar to the configuration of dynamic traps. But, the key difference is that the trap action must not be specified.

### Rate Limit Data Path Traffic

To set the rate limit (bytes/bits per second) and burst size for a specific packet type, use the following command:

```
tc ... action police rate RATE burst BYTES conform-exceed drop [action pass]
```

In this command:

- **RATE:** Specifies the rate limit in bits or bytes per second.
- **BYTES:** Indicates the burst size in bytes.

**Note:** The `action pass` is the default action and can be omitted.

**Example Configuration:**

To limit the rate of traffic matching a specific source MAC address, follow these steps:

First, add a Qdisc and a filter to the ingress of the network interface `swp26`. This limits the rate of incoming traffic matching the specified source MAC address.

```
tc qdisc add dev swp26 clsact
tc filter add dev swp26 ingress flower skip_sw src_mac 00:b5:4d:b1:32:22 action police rate 100mbps burst 64000 conform-exceed drop
```

To verify that the configuration has been applied successfully for ingress traffic, use the following command:

```
tc filter show dev swp26 ingress
```

Output:

```
filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:22
  skip_sw
  in_hw in_hw_count 1
        action order 1:  police 0x4 rate 800Mbit burst 64000b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

Also, you can apply the same rate limiting configuration to egress traffic by adding a filter to the egress of the same network interface.

```
tc filter add dev swp26 egress flower skip_sw src_mac 00:b5:4d:b1:32:22 action police rate 100mbps burst 64000 conform-exceed drop
```

To verify that the configuration has been applied successfully for egress traffic, use the following command:

```
tc filter show dev swp26 egress
```

Output:

```
filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:22
  skip_sw
  in_hw in_hw_count 1
        action order 1:  police 0x5 rate 800Mbit burst 64000b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

**Note:** You can set the maximum policing rate to the same rate as for dynamic rules, but the actual rate is limited by the capabilities of the physical port.

### Assigning a Policer to Multiple Ports

Similar to dynamic traps, you can use a shared block to police data path traffic for multiple ports. In this scenario, the expected rate is divided among all ports.

**Example Configuration:**

To illustrate, let's assign a policer to two ports (`swp27` and `swp28`) for traffic matching a specific source MAC address. Follow these steps:

First, add ingress blocks with a shared block ID (`1`) to the respective network interfaces:

```
tc qdisc add dev swp27 ingress_block 1 clsact
tc qdisc add dev swp28 ingress_block 1 clsact
```

Next, add a filter to the shared block (`block 1`) for both ports. This filter will police traffic matching the specified source MAC address:

```
tc filter add block 1 ingress flower skip_sw src_mac 00:b5:4d:b1:32:24 action police rate 1mibps burst 6000 conform-exceed drop
```

To verify that the configuration has been applied successfully for ingress traffic, use the following command for each port:

```
tc filter show dev swp27 ingress
```

Output:

```
filter block 1 protocol ip pref 20 flower chain 0
filter block 1 protocol ip pref 20 flower chain 0 handle 0x1
  eth_type ipv4
  dst_ip 192.168.0.0/16
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 1 ref 1 bind 1
        used_hw_stats delayed

filter block 1 protocol all pref 49151 flower chain 0
filter block 1 protocol all pref 49151 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:24
  skip_sw
  in_hw in_hw_count 1
        action order 1:  police 0x6 rate 8388Kbit burst 5998b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed

filter block 1 protocol all pref 49152 flower chain 0
filter block 1 protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:24
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 5 ref 1 bind 1
        used_hw_stats delayed

        action order 2:  police 0x2 rate 8388Kbit burst 5998b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

Similar output would be shown for `swp28` as well.

Optionally, you can apply the same rate limiting configuration to egress traffic by adding egress blocks with a shared block ID (`2`) to the respective network interfaces:

```
tc qdisc add dev swp23 egress_block 2 clsact
tc qdisc add dev swp24 egress_block 2 clsact
```

Then, add a filter to the egress shared block (`block 2`) for both ports:

```
tc filter add block 2 egress flower skip_sw src_mac 00:b5:4d:b1:32:24 action police rate 1mibps burst 6000 conform-exceed drop
```

To verify that the configuration has been applied successfully for egress traffic, use the following command for each port:

```
tc filter show dev swp23 egress
```

Output:

```
filter block 2 protocol all pref 49152 flower chain 0
filter block 2 protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:24
  skip_sw
  in_hw in_hw_count 1
        action order 1:  police 0x7 rate 8388Kbit burst 5998b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

Similar output would be shown for `swp24` as well.

**Note:**

- The expected rate is divided among all ports in the shared block.
- The same rate limiting configuration can be applied to both ingress and egress traffic if needed.

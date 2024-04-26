---
title: Policing of Dynamic Traps
grand_parent: Network Configuration
parent: Traffic Control and Policing
nav_order: 1
layout: default
---

# Policing of Dynamic Traps

Policing in networking is all about managing data traffic effectively. It's like being a traffic cop for information flowing through a network.

This section explores extending ACLs to trap packets dynamically, using tools like `tc`, enabling customized rate limits and traffic classes tailored to specific network requirements.

### Rate Limit of a Trapped Packet

To control the rate at which trapped packets are processed, you can configure the rate limit and burst size using the `tc` policer action as part of creating an ACL rule.

To set the rate limit and burst size for trapping a packet, use the following `tc` command:

```
tc ... action trap action police rate RATE burst BYTES conform-exceed drop
```

In this command:

- **RATE:** Specifies the rate limit in bits or bytes per second. You can use various units such as bits per second (bit, kbit, mbit, etc.) or bytes per second (bps, kbps, mbps, etc.). For example, you can set a rate limit of 1 megabit per second as `rate 1mbit`.

- **BYTES:** Indicates the maximum allowed burst size in bytes. This determines the amount of data that can be transmitted in short bursts.

**Note:** The `police` action requires both the `rate` and `burst` parameters. Other parameters are ignored.

Packets that exceed the configured bandwidth limit are dropped, regardless of the `conform-exceed` policer parameter setting defined.

**Example Configuration:**

To illustrate how to trap packets based on a specific source MAC address and configure rate limits, let's consider an example where we want to trap packets with the source MAC address `00:B5:4D:B1:32:22`. We'll set the rate limit to `1024K bytes-per-second` and the burst size to `2096 bytes`.

First, we need to add a Qdisc and a filter to the network interface. Enter the following commands:

```
tc qdisc add dev swp30 clsact
tc filter add dev swp30 ingress flower skip_sw src_mac 00:B5:4D:B1:32:22 action trap action police rate 1mibps burst 2096 conform-exceed drop
```

To verify that the configuration has been applied successfully, use the following command:

```
tc filter show dev swp30 ingress
```

Output:

```
filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:b5:4d:b1:32:22
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 3 ref 1 bind 1
        used_hw_stats delayed

        action order 2:  police 0x1 rate 8388Kbit burst 2094b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

**Note:**

- The rule and policer of the packet are bound to one port only (`swp30` in this example).
- To bind the rule and the policer to multiple ports, use the shared block feature of the `tc` tool. Refer to the [ACL-support](https://docs.dent.dev/NetworkConfigurations/NetworkAddressingAndFilteringSubCategories/ACL-Support.html) documentation for detailed instructions on configuring a shared block.

### Setting the TC of a Trapped Packet

To configure the Traffic Class (TC) of a user-defined trap, you can use the `hw_tc` option of the TC flower filter. This option allows you to specify the TC queue number (TCID) for the trapped packet. If the `hw_tc` option is not specified, the default TC value of 3 is used for all ACL traps.

Use the following syntax to set the TC of a trapped packet:

```
tc ... hw_tc TCID action trap ...
```

In this command:

- **TCID:** Specifies the TC queue number, ranging from 0 to 7.

By setting the TC of a trapped packet, you can prioritize its handling within the network according to your specific requirements and traffic management policies.

**Example Configuration:**

To illustrate how to set the Traffic Class (TC) of a trapped packet, let's consider an example where we want to trap packets with the source MAC address `00:B5:4D:B1:32:22` and set the TC of that trap to the highest value (7).

First, add a Qdisc and a filter to the network interface. Enter the following commands:

```
tc qdisc add dev swp31 clsact
tc filter add dev swp31 ingress flower skip_sw src_mac 00:B5:4D:B1:32:22 hw_tc 7 action trap
```

To verify that the configuration has been applied successfully, use the following command:

```
tc filter show dev swp31 ingress
```

Output:

```
filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1 hw_tc 7
  src_mac 00:b5:4d:b1:32:22
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 4 ref 1 bind 1
        used_hw_stats delayed
```

This output confirms that the filter has been successfully applied with the TC set to 7 for packets matching the specified source MAC address.

### Assigning a Policer to Multiple Ports

You can assign a policer ACL rule to multiple ports using shared blocks, similar to assigning a generic ACL rule. This allows you to apply the same policer configuration across multiple network ports.

**Example Configuration:**

To illustrate, let's assign a trap ACL rule to two ports (`swp32` and `swp33`) and set a rate limit for the traffic flow. Follow these steps:

First, add Qdiscs with ingress blocks to the respective network interfaces:

```
tc qdisc add dev swp32 ingress_block 1 clsact
tc qdisc add dev swp33 ingress_block 1 clsact
```

Next, add a filter to the shared block (`block 1`) for both ports. This filter will trap packets with a specific source MAC address and apply a policer to them:

```
tc filter add block 1 ingress flower skip_sw src_mac 00:B5:4D:B1:32:24 action trap action police rate 1mibps burst 6000 conform-exceed drop
```

To verify that the configuration has been applied successfully, use the following command for each port:

```
tc filter show dev swp32 ingress
```

```
tc filter show dev swp33 ingress
```

Output for `swp32`:

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

Output for `swp33` would be similar, showing the filter configuration for the shared block `block 1` on that port.

This configuration ensures that packets matching the specified source MAC address on both `swp32` and `swp33` ports are trapped and subjected to the configured policer rules.

**Note:** Optionally, you can use the `hw_tc` option to specify the TC queue number for the trapped packets, if needed.

### Precedence Over Static Trap Limits

When adding a dynamic (user) ACL rule that matches a static profile flow, the user-defined rule takes precedence over the static rule. All limitations on dynamic traps are applied for this trap, such as a maximum of 4000 packets per second (pps) per Traffic Class (TC) and a default TC value of 3.

**Example Configuration:**

To illustrate, let's create an ACL trap rule that takes precedence over the LLDP static rate limiter. Follow these steps:

First, add a Qdisc and a filter to the network interface `swp25`. Enter the following commands:

```
tc qdisc add dev swp25 clsact
tc filter add dev swp25 ingress flower skip_sw src_mac 01:80:c2:00:00:0e action trap action police rate 1mibps burst 2096 conform-exceed drop
```

To verify that the configuration has been applied successfully, use the following command:

```
tc filter show dev swp25 ingress
```

Output:

```
filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 01:80:c2:00:00:0e
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 6 ref 1 bind 1
        used_hw_stats delayed

        action order 2:  police 0x3 rate 8388Kbit burst 2094b mtu 2Kb action drop overhead 0b
        ref 1 bind 1
        used_hw_stats delayed
```

**Additional Notes:**

- It is also possible to change the default TC value in the example above.
- Supported parameters for the policer action include rate, burst, and conform-exceed.
- Specifying rate as a percentage is not supported.
- Specifying cell size for the burst option is not supported.
- Specifying burst values less than the packet size sent to the port does not drop all packets. Instead, packets are rate-limited according to the user-defined rate.

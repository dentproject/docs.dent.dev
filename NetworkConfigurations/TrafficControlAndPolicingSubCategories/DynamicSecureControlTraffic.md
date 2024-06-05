---
title: Dynamic Secure Control Traffic
grand_parent: Network Configuration
parent: Traffic Control and Policing
nav_order: 4
layout: default
---

# Dynamic Secure Control Traffic

Dynamic Secure Control Traffic (SCT) configuration is crucial in protecting the CPU from being overwhelmed by the traffic it must process. This mechanism limits the amount of traffic processed by the CPU by configuring limits on a per-group basis using packets-per-second (pps) values.

## Initial SCT Values

An initial configuration is set by the driver upon initiation, but users can revise this configuration.

The initial SCT values for various traffic types are as follows:

|     Traffic Type     | TC (Queue) | Rate (pps) |
| :------------------: | :--------: | :--------: |
|         BGP          |     7      |    1000    |
| All-Routers MC (BGP) |     7      |    100     |
|       STP BPDU       |     7      |    200     |
|         LACP         |     7      |    200     |
|         VRRP         |     7      |    200     |
|         OSPF         |     7      |    1000    |
|         ISIS         |     7      |    1000    |
|         LLDP         |     6      |    200     |
|      802.1X PAE      |     6      |    200     |
|         CDP          |     6      |    200     |
|         SSH          |     5      |    1000    |
|        Telnet        |     5      |    200     |
|       DHCP BC        |     4      |    100     |
|         ICMP         |     4      |    100     |
|   ARP reply to me    |     4      |    300     |
|        ARP BC        |     4      |    100     |
|         IGMP         |     4      |    400     |
|   IP to My address   |     2      |   10000    |
|        IP BC         |     2      |    100     |
|   IP route default   |     1      |    400     |
|      All other       |     0      |    100     |
|   ACL default trap   |    0-7     |    4000    |

## User Configuration

Users can configure rate limiting (pps) for specified packet types/groups through a set of temporary debugfs interfaces. These interfaces are located under the root of the debugfs mount point, within the `prestera/sct/` subfolder.

### Reading Configuration

To read the current SCT configuration, use the `ls` command:

```
ls /sys/kernel/debug/prestera/sct/
```

This command will list the available SCT files:

```
all_unspecified_cpu_opcodes
sct_acl_trap_queue_4
sct_arp_to_me
sct_dhcp
sct_isis
sct_special_ip4_icmp_redirect
sct_stp
sct_acl_trap_queue_0
sct_acl_trap_queue_5  sct_bgp
sct_icmp
sct_lacp  sct_special_ip4_mtu_exceed
sct_telnet
sct_acl_trap_queue_1
sct_acl_trap_queue_6
sct_bgp_all_routers_mc
sct_igmp
sct_lldp
sct_special_ip4_options_in_ip_hdr
sct_vrrp
sct_acl_trap_queue_2
sct_acl_trap_queue_7
sct_cdp
sct_ip_bc
sct_nat
sct_special_ip4_zero_ttl
sct_acl_trap_queue_3
sct_arp_intervention
sct_default_route
sct_ip_to_me
sct_ospf
sct_ssh
```

### Writing Configuration

To set a custom rate for a specific group, use the `echo` command. For example, to set the SCT rate for SSH traffic to 200 pps:

```
echo 200 > /sys/kernel/debug/prestera/sct/sct_ssh
```

To verify the new setting, use the `cat` command:

```
cat /sys/kernel/debug/prestera/sct/sct_ssh
```

Output:

```
sct_ssh: 200 (pps)
```

### Disabling SCT

To disable SCT for a specific group, set its value to `0`. This action automatically sets the value to `65535` (disabling the limit):

```
echo 0 > /sys/kernel/debug/prestera/sct/sct_ssh
```

To verify the setting, use the `cat` command:

```
cat /sys/kernel/debug/prestera/sct/sct_ssh
```

Output:

```
sct_ssh: 65535 (pps)
```

### Notes

- The maximum SCT value that can be set is `65K` pps.
- Setting an SCT group limit value to zero effectively disables the limit by setting it to `65535`.

## Verify Configuration

Let's say you want to limit SSH traffic to different pps values and test it using iperf on the same machine:

### Enable SCT for SSH:

```
echo <new_limit> > /sys/kernel/debug/prestera/sct/sct_ssh
```

Example:

```
# Limit SSH traffic to 200 pps
echo 200 > /sys/kernel/debug/prestera/sct/sct_ssh
```

### Start iperf Server for SSH Traffic:

```
iperf -s -p 23
```

### Start iperf Client:

```
iperf -c 127.0.0.1 -p 23
```

Output-

```
------------------------------------------------------------
Client connecting to 127.0.0.1, TCP port 23
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 43796 connected with 127.0.0.1 port 23
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  11.7 GBytes  10.0 Gbits/sec
```

By following this approach, we can efficiently test a range of SCT limits for SSH traffic and provide valuable information on optimizing SCT configurations based on specific requirements and network environments.

### Test Results:

Here's a summary of the observed bandwidth for each SCT limit:

| SCT Limit (pps) | Observed Bandwidth (Gbps) |
| :-------------: | :-----------------------: |
|       100       |           9.94            |
|       200       |           10.5            |
|       300       |           10.7            |
|       400       |           10.8            |
|       500       |           10.0            |
|       600       |           10.3            |
|       700       |           9.83            |
|       800       |           10.4            |
|       900       |           11.0            |
|      1000       |           10.0            |

### Analysis:

- Lower SCT limits (e.g., 100-500 pps) appear to have a minimal impact on bandwidth, with fluctuations within a relatively narrow range.
- Higher SCT limits (e.g., 600-900 pps) result in slightly higher bandwidth, peaking at around 11.0 Gbps at 900 pps.
- At the highest SCT limit tested (1000 pps), the observed bandwidth decreases slightly to 10.0 Gbps.

Based on the observed results, users can select SCT limits that strike a balance between traffic control and maximizing network throughput according to their specific requirements.

---
title: Traffic Shaping
grand_parent: Network Configuration
parent: Traffic Control and Policing
nav_order: 3
layout: default
---

# Traffic Shaping

Traffic shaping is a technique used to control the flow and volume of network traffic, ensuring optimal performance and efficient utilization of network resources.

In DENT, traffic shaping is configured using the `tc` (traffic control) command, which manages Queuing Disciplines (qdiscs). There are two main types of qdiscs: classful and classless.

### Classful Queuing Disciplines

Classful qdiscs provide flexibility and control over network traffic by allowing filters to be attached to them. These filters enable packets to be directed to specific classes and subqueues. The structure of classful qdiscs can be visualized as a tree, with the root qdisc at the top and various classes and subqueues branching out from it.

**Key Terminology**

- **Root Classes:** Classes directly attached to the root qdisc.
- **Inner Classes:** Generic term for classes attached to the root qdisc.
- **Leaf Classes:** Terminal classes in a queuing discipline, analogous to the leaves of a tree.
- **Family Relationships:** Figurative language depicting the hierarchical structure of classes.

**Benefits**

- Allows for complex configurations and fine-grained control over traffic.
- Enables the use of filters to direct packets to specific classes, ensuring precise traffic management.

For more detailed information on classful qdiscs, refer to the [Classful Qdiscs HOWTO](https://tldp.org/HOWTO/Traffic-Control-HOWTO/classful-qdiscs.html).

### Classless Queuing Disciplines

Classless qdiscs can be used either as the primary qdisc on an interface or within a leaf class of a classful qdisc. These are the fundamental schedulers used, with `pfifo_fast` being the default scheduler.

**Characteristics**

- **Primary Qdisc:** Can be applied directly to an interface to manage traffic.
- **Leaf Class Integration:** Can be used within a leaf class of a classful qdisc for additional scheduling.

**Benefits**

- Easier to configure compared to classful qdiscs.
- Provides fundamental scheduling without the need for complex configurations.

For more detailed information on classless qdiscs, refer to the [Classless Qdiscs HOWTO](https://tldp.org/HOWTO/Traffic-Control-HOWTO/classless-qdiscs.html).

## Common Qdiscs and Their Uses

## ETS (Enhanced Transmission Selection) Qdisc

- **Use:** Scheduling configuration.
- **Purpose:** Ensures that different traffic types are allocated appropriate bandwidth and priorities.

In DENT, a scheduler with strict and WRR (Weighted Round Robin) bands is implemented using the ETS queuing discipline. ETS provides enhanced control over traffic transmission by allowing strict and weighted prioritization of traffic. For detailed information on the ETS qdisc, refer to the [tc-ets(8)](https://man7.org/linux/man-pages/man8/tc-ets.8.html).

### Priomap

ETS supports several traffic classification algorithms, but only priomap is offloaded. Priomap is a critical component of ETS configuration, determining the mapping of packet priorities to specific bands for transmission.

It consists of a list of numbers, each corresponding to a priority level. These numbers indicate the band number that packets with that priority should be directed to, with 0 representing the first band, 1 for the second, and so forth.

```
                                    p7 ----------------.
                                    ..                 |
                                    p2 ------.         |
                                    p1 ----. |   ...   |
                                    p0 --. | |         |
                                        | | |         |
                                        v v v         v
                tc ... ets bands 8 priomap 7 6 5 4 3 2 1 0
                                        | | |   ...   |
                                        | | |         '-> band 0
                                        | | |              ...
                                        | | '-----------> band 5
                                        | '-------------> band 6
                                        '---------------> band 7
```

**Key Points:**

- ETS supports up to 16 priorities in a priomap. However, for the purposes of offloading in DENT NOS, only priorities 0-7 are relevant. Priorities 8-15 are ignored and can be omitted when configuring ETS.
- The mapping of a priority to a traffic class is static and cannot be altered dynamically. Attempting to change the priority mapping results in the qdisc not being offloaded, and the offload flag is not set.

To view the current ETS configuration, including the full priomap and statistics, the `tc -s show` command can be used. This command provides detailed insight into the current ETS setup, allowing administrators to monitor traffic classification and band allocation effectively.

### Band Number Mapping

In DENT, bands are used to denote logical traffic classes, with each band mapped to a pair of Traffic Classes (TCs) in the ASIC. The mapping of bands to TCs is as follows: band 0 maps to TC 7, band 1 to TC 6, and so forth, with band 7 mapping to TC 0.

| Band number | Class ID | Priority |
| :---------: | :------: | :------: |
|      0      |   X:1    | Highest  |
|      1      |   X:2    |          |
|      2      |   X:3    |          |
|      3      |   X:4    |          |
|      4      |   X:5    |          |
|      5      |   X:6    |          |
|      6      |   X:7    |          |
|      7      |   X:8    |  Lowest  |

**Example Configuration:**

To illustrate, consider the following example of adding an ETS qdisc with 8 bands:

```
tc qdisc replace dev swp21 root handle 10: ets bands 8 strict 4 quanta 4000 3000 2000 1000 priomap 7 6 5 4 3 2 1 0
```

Output:

```
qdisc ets 10: root refcnt 2 offloaded bands 8 strict 4 quanta 4000 3000 2000 1000 priomap 7 6 5 4 3 2 1 0 7 7 7 7 7 7 7 7
```

This command creates an ETS qdisc with 8 bands, comprising of:

- 4 strict bands
- 4 bands that split the traffic in a ratio of 40% : 30% : 20% : 10%

Traffic is mapped to bands in a reversed 1:1 manner to prioritize higher numbered bands. Priority 0 traffic is directed to TC 0, priority 1 to TC 1, and so on. However, BUM (Broadcast, Unknown unicast, and Multicast) traffic is directed to TCs 8, 9, and so forth.

When creating a new qdisc to attach to a band, use the corresponding class ID as the parent reference. For example, to attach RED to the first band and TBF to the second one:

```
tc qdisc add dev swp23 root handle 10: htb
tc class add dev swp23 parent 10: classid 10:1 htb rate 100mbit

tc qdisc replace dev swp23 parent 10:1 handle 101: red limit 2M avpkt 1000 probability 0.1 min 500K max 1.5M
```

This configures RED (Random Early Detection) qdisc with specified parameters and attaches it to the first band. Similarly, TBF (Token Bucket Filter) qdisc can be attached to other bands as required.

```
RED: set bandwidth to 10Mbit
```

The output confirms that the bandwidth for RED qdisc has been successfully set to 10Mbit.

```
tc class show dev swp23
```

```
class htb 10:1 root leaf 101: prio 0 rate 100Mbit ceil 100Mbit burst 1600b cburst 1600b
class red 101:1 parent 101:
```

The output of `tc class show` command displays the configured classes, including the htb (Hierarchy Token Bucket) class with rate and ceiling set to 100Mbit, and the RED class.

```
tc -s qdisc show dev swp23
```

```
qdisc htb 10: root refcnt 2 r2q 10 default 0 direct_packets_stat 0 direct_qlen 1000
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
qdisc red 101: parent 10:1 limit 2Mb min 500Kb max 1536Kb
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
  marked 0 early 0 pdrop 0 other 0
qdisc clsact ffff: parent ffff:fff1
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
```

The output of `tc -s qdisc show` command provides detailed information about the configured qdiscs, including statistics and settings for each. In this case, it shows the HTB and RED qdiscs configured on the device `swp23`.

Verfication using `iperf` utility:

```
# Run iperf to verify traffic shaping with 15Mb/s
iperf -c 127.0.0.1 -t 60 -b 15M
```

Please make sure to run your iperf server using `iperf -s` command.

Output-

```
------------------------------------------------------------
Client connecting to 127.0.0.1, TCP port 5001
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 52146 connected with 127.0.0.1 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-60.0 sec  71.5 MBytes  10.00 Mbits/sec
```

When testing with `iperf` to send 15Mb/s of traffic, the output shows a bandwidth of 10Mb/s, demonstrating that the traffic shaping configuration effectively limits the bandwidth to the configured rate of 10Mb/s.

Consider another example where a Token Bucket Filter (TBF) qdisc is attached to the second class of an Hierarchy Token Bucket (HTB) qdisc:

```
tc qdisc add dev swp24 root handle 10: htb default 1
tc class add dev swp24 parent 10: classid 10:2 htb rate 1Gbit

tc qdisc replace dev swp24 parent 10:2 handle 102: tbf rate 400Mbit burst 128K limit 1M
```

This configures an HTB qdisc on device `swp24` with a default class (`10:1`) and a second class (`10:2`) with a rate of 1Gbit. Then, a TBF qdisc is attached to the second class with a rate of 400Mbit, burst size of 128K, and a limit of 1M.

```
tc qdisc show dev swp24
```

```
qdisc tbf 102: parent 10:2 rate 400Mbit burst 131050b lat 18.4ms
```

The output of `tc qdisc show` command displays the HTB and TBF qdiscs configured on device `swp24`, indicating their respective settings such as rates, burst sizes, and latencies.

```
tc class show dev swp24
```

```
class htb 10:2 root leaf 102: prio 0 rate 1Gbit ceil 1Gbit burst 1375b cburst 1375b
class tbf 102:1 parent 102:
```

The output of `tc class show` command shows the configured classes, including the HTB and TBF classes. It displays settings such as rates, burst sizes, and ceiling rates.

```
tc -s qdisc show dev swp24
```

```
qdisc htb 10: root refcnt 2 r2q 10 default 0x1 direct_packets_stat 0 direct_qlen 1000
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
qdisc tbf 102: parent 10:2 rate 400Mbit burst 131050b lat 18.4ms
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
```

The output of `tc -s qdisc show` command provides detailed information about the configured qdiscs, including statistics and settings for each. In this case, it shows the HTB and TBF qdiscs configured on the device `swp24`.

Verfication using `iperf` utility:

```
# Run iperf to verify traffic shaping with 500Mbits/sec
iperf -c 127.0.0.1 -t 60 -b 500M
```

Please make sure to run your iperf server using `iperf -s` command.

Output-

```
------------------------------------------------------------
Client connecting to 127.0.0.1, TCP port 5001
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 51982 connected with 127.0.0.1 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-60.0 sec  2.80 GBytes  400 Mbits/sec
```

When testing with `iperf` to send 500Mbit of traffic, the output shows a bandwidth of 400Mbit, demonstrating that the TBF configuration effectively limits the bandwidth to 400Mbit.

### Default Behavior

By default, the Scheduler for all traffic classes on all ports is configured to be SDWRR (Strict Deficit Weighted Round Robin) upon switch initialization. This default behavior ensures fair handling of traffic across different classes and ports.

### Default TC Scheduler Configuration

The default TC scheduler configuration assigns SDWRR weights to each traffic class, as shown in the following table:

| TC Band | TC  | SDWRR Weight |
| :-----: | :-: | :----------: |
|    1    |  7  |      5       |
|    2    |  6  |      5       |
|    3    |  5  |      5       |
|    4    |  4  |      5       |
|    5    |  3  |      1       |
|    6    |  2  |      1       |
|    7    |  1  |      1       |
|    8    |  0  |      1       |

### Default Qdisc Configuration

When a network interface is created, the operating system automatically applies a default scheduling profile (qdisc) on that interface. Similarly, when the last Traffic Control (TC) qdisc is removed from a port, the default qdisc configuration is automatically applied.

This ensures that every network interface always has at least one (default) queuing discipline (qdisc) set, maintaining proper traffic handling even in the absence of explicit configurations.

**Retrieving Default Qdisc Configuration:**

You can obtain the default qdisc configuration using either of the following commands:

```
cat /proc/sys/net/core/default_qdisc
```

Output:

```
pfifo_fast
```

```
sysctl net.core.default_qdisc
```

Output:

```
net.core.default_qdisc = pfifo_fast
```

### Modifying Default Qdisc Configuration:

To change the default qdisc configuration, you can use the corresponding `sysctl` command or `echo` command, depending on your system requirements and preferences.

This default qdisc configuration ensures that network interfaces are properly configured for traffic management, even without explicit user interventions.

### Limitations

- ETS supports up to 16 different scheduler profiles, with one reserved for default configuration.
- The user has to specify 8 queues when creating a qdisc.
- The mapping of priority to band is static and cannot be changed.
- Default qdiscs applied on network interfaces are not offloaded.

## TBF (Token Bucket Filter) Qdisc

- **Use:** Shaper configuration.
- **Purpose:** Controls the rate of traffic by shaping the output flow, smoothing out bursts and maintaining a steady rate.

In DENT, the Shaper is implemented using a Token Bucket Filter queuing discipline. If the implementation is supported, it can be offloaded to hardware. For more information, refer to [tc-tbf(8)](https://man7.org/linux/man-pages/man8/tc-tbf.8.html).

On hardware, shaping is implemented using a token bucket. TBF is configured on the ETS qdisc bands. The following TBF parameters are offloaded:

- **rate**: The speed with which the queued traffic will be sent.
- **burst**: The number of bytes of traffic that is dequeued before the shaper rate takes effect.

### Hardware Statistics

Hardware statistics are also supported. To see the statistics, use the show command that shows the current configuration, including TBF's statistics:

```
tc -s qdisc show dev <interface>
```

**Example Configuration:**

The following example attaches a TBF qdisc under band 0 of an ETS parent whose handle is 101. It configures a 400Mbps shaper with a burst size of 128KiB:

```
tc qdisc add dev swp25 root handle 10: htb default 1
tc class add dev swp25 parent 10: classid 10:1 htb rate 1Gbit

tc qdisc add dev swp25 parent 10:1 handle 101: tbf rate 400Mbit burst 128K limit 1M
```

Output:

```
tc qdisc show dev swp25
```

```
qdisc htb 10: root refcnt 2 r2q 10 default 0x1 direct_packets_stat 0 direct_qlen 1000
```

The qdisc htb output indicates the root queuing discipline's configuration, with values such as a reference count of 2, queuing limits set with a direct queue length of 1000 packets, and no packets sent or dropped, reflecting the current state of the root queue.

```
tc class show dev swp25
```

```
class htb 10:1 root prio 0 rate 1Gbit ceil 1Gbit burst 1375b cburst 1375b
```

In the class htb output, the configured hierarchical token bucket class exhibits parameters like a priority of 0, a rate set to 1Gbit, and burst sizes of 1375 bytes for both the configured and committed burst, showcasing the root class's traffic shaping characteristics.

```
tc -s qdisc show dev swp25
```

```
qdisc htb 10: root refcnt 2 r2q 10 default 0x1 direct_packets_stat 0 direct_qlen 1000
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
qdisc tbf 101: parent 10:1 rate 400Mbit burst 131050b lat 18.4ms
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
```

The qdisc tbf output reveals specific details about the Token Bucket Filter settings under the root queuing discipline, including a rate of 400Mbit, a burst size of 131050 bytes, and a latency of 18.4 milliseconds, providing insights into traffic shaping at the root level.

Verfication using `iperf` utility:

```
# Run iperf to verify traffic shaping with 500Mbits/sec
iperf -c 127.0.0.1 -t 60 -b 500M
```

Please make sure to run your iperf server using `iperf -s` command.

Output-

```
------------------------------------------------------------
Client connecting to 127.0.0.1, TCP port 5001
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 51824 connected with 127.0.0.1 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-60.0 sec  2.60 GBytes  400 Mbits/sec
```

When testing with `iperf` to send 500Mbps of traffic, the output shows a bandwidth of 400Mbps, demonstrating that the TBF configuration effectively limits the bandwidth to 400Mbps.

### Limitations

- Max burst value that can be set is 16M and should be a multiple of 4k.
- The hardware may adjust the rate value set by the user. The actual value is not shown in the show command.

## RED (Random Early Detection) Qdisc

- **Use:** Congestion avoidance.
- **Purpose:** Prevents congestion by randomly dropping packets early, before the queue becomes full.

In DENT, congestion avoidance is implemented using the RED queuing discipline, configured on the ETS qdisc bands. For more information, see [tc-red(8)](https://man7.org/linux/man-pages/man8/tc-red.8.html).

On the hardware, the algorithm is implemented using a WRED (Weighted Random Early Detection) mechanism.

The following parameters of the RED qdisc are offloaded:

- **min**: The minimum queue size.
- **max**: The maximum limit.
- **probability**: The probability of dropping a packet when the average queue size reaches the maximum limit. A probability of 1.0 means 100%.

**Note**: Other parameters are not offloaded, but they may be required to be provided by the user.

To see RED statistics on each band (ETS bands and its child RED qdiscs), use the `qdisc show` command on the port. This command displays the current qdisc configuration, including queue statistics:

```
tc -s qdisc show dev <interface>
```

**Example Configuration:**

The following example attaches a RED qdisc under band 0 of an ETS parent whose handle is 10. Between the queue depths of 500KiB and 1.5MiB, the probability of dropping packets gradually rises from 0 to 10%.

```
tc qdisc add dev swp26 root handle 10: htb default 1
tc class add dev swp26 parent 10: classid 10:1 htb rate 1Gbit

tc qdisc add dev swp26 parent 10:1 handle 101: red limit 2M avpkt 1000 probability 0.1 min 500K max 1.5M
```

```
RED: set bandwidth to 10Mbit
```

The output confirms that the bandwidth for RED qdisc has been successfully set to 10Mbit.

To view the current qdisc configuration and queue statistics:

```
tc -s qdisc show dev swp26
```

```
qdisc htb 10: root refcnt 2 r2q 10 default 0x1 direct_packets_stat 0 direct_qlen 1000
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
qdisc red 101: parent 10:1 limit 2Mb min 500Kb max 1536Kb
 Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
 backlog 0b 0p requeues 0
  marked 0 early 0 pdrop 0 other 0
```

The output displays the RED qdisc configuration under the HTB parent, including the limit, min and max queue sizes, and drop probabilities.

Verfication using `iperf` utility:

```
# Run iperf to verify traffic shaping with 3.5Mbits/sec
iperf -c 127.0.0.1 -t 60 -b 3.5M
```

Please make sure to run your iperf server using `iperf -s` command.

Output-

```
------------------------------------------------------------
Client connecting to 127.0.0.1, TCP port 5001
TCP window size: 2.50 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 51748 connected with 127.0.0.1 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  12.1 MBytes  2.00 Mbits/sec
```

The test successfully demonstrated bandwidth shaping with `iperf`, showcasing effective enforcement close to the configured 2Mbps limit, despite the traffic sent at 3.5Mbps.

### Limitations

- Only 16 different RED profiles (one per port) are available on the system.
- Profiles are not shared among ports.
- At least one profile is reserved for default configuration (when the RED qdisc is removed).
- On some platforms, more profiles are reserved for internal purposes.
- Congestion avoidance configuration for multicast traffic is not supported.

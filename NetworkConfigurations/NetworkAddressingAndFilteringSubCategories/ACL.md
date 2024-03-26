---
title: Access Control Lists
grand_parent: Network Configuration
parent: Network Addressing and Filtering
nav_order: 2
layout: default
---

# Access Control Lists

Access Control Lists (ACLs) empower you to finely control traffic by defining specific criteria, such as the 5-tuple, and corresponding actions like pass or drop. These rules serve various purposes including restricting traffic flow, managing traffic rates, tracking statistics, and initiating network address translation.

- It was introduced in the [Arthur Release (v1)](https://github.com/dentproject/dentOS/releases/tag/v1.0) and was further refined in later DENT versions.
- They offer flexibility as they can be dynamically added or removed at any time.

ACLs can be applied to both inbound and outbound traffic at the port level, ensuring comprehensive network management.

### ACL Configuration

**Add ACL rules**

Before configuring match rules on DENT, it is essential to first establish the queuing disciplines (qdiscs) to which the flower classifier will be linked.

To apply an ingress qdisc or a clsact qdisc to the port, execute the following tc command:

```
tc qdisc add dev DEV-NAME {ingress|clsact}
```

Replace "DEV-NAME" with the appropriate switchdev interface name for the port you are configuring, such as "swp30". This command allows you to set up either an ingress qdisc or a clsact qdisc on the specified network device.

To create ingress queuing disciplines (qdiscs) for `swp30` and `swp31`, you can use the following commands:

```
tc qdisc add dev swp30 ingress
tc qdisc add dev swp31 clsact
```

These commands set up an ingress qdisc for `swp30` and a clsact qdisc for `swp31`.

To list the existing qdiscs, you can use:

```
tc qdisc show
```

Upon running this command, you will observe the following output:

```
qdisc ingress ffff: dev swp30 parent ffff:fff1 ----------------
qdisc clsact ffff: dev swp31 parent ffff:fff1
```

This output confirms the successful creation of the ingress qdisc for `swp30` and the clsact qdisc for `swp31`.

The ACL rule configuration utilizes the following format when executed with `tc`:

```
Usage:  tc [ OPTIONS ] OBJECT { COMMAND | help }
        tc [-force] -batch filename
where  OBJECT := { qdisc | class | filter | chain |
                    action | monitor | exec }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[aw] |
                    -o[neline] | -j[son] | -p[retty] | -c[olor]
                    -b[atch] [filename] | -n[etns] name | -N[umeric] |
                     -nm | -nam[es] | { -cf | -conf } path
                     -br[ief] }
```

**_Note_**: The `-brief` option is present for DENT Hardware but missing in the case of DENT VM.

Once the qdisc is created, you can add flower rules that are bound to a specific qdisc or switchdev interface. These rules can be defined for software, hardware, or both.

- To define a rule for software only, include the `skip_hw` parameter.
- To define a rule for hardware only, include the `skip_sw` parameter.
- To define a rule for both software and hardware, omit these parameters.

To create a flower rule that drops an IP packet with the source address 192.168.1.1 on ingress:

```
tc filter add dev swp30 ingress protocol ip pref 10 flower skip_sw src_ip 192.168.1.1 action drop
```

```
filter protocol ip pref 10 flower chain 0
filter protocol ip pref 10 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.1
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 2 ref 1 bind 1
        used_hw_stats delayed
```

To create a flower rule that drops egress IP packets with the source address 192.168.1.2:

```
tc filter add dev swp30 egress protocol ip pref 10 flower skip_sw src_ip 192.168.1.2 action drop
```

```
filter protocol ip pref 10 flower chain 0
filter protocol ip pref 10 flower chain 0 handle 0x2
  eth_type ipv4
  src_ip 192.168.1.2
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 5 ref 1 bind 1
        used_hw_stats delayed
```

To add a rule to trap-to-CPU:

```
tc filter add dev swp30 ingress protocol ip pref 30 flower skip_sw src_ip 192.168.1.3 action trap
```

```
filter protocol ip pref 30 flower chain 0
filter protocol ip pref 30 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.3
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 6 ref 1 bind 1
        used_hw_stats delayed
```

To display qdiscs filter rules:

```
tc filter show dev swp30 ingress
```

```
filter protocol ip pref 10 flower chain 0
filter protocol ip pref 10 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.1
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 2 ref 1 bind 1
        used_hw_stats delayed

filter protocol ip pref 10 flower chain 0
filter protocol ip pref 10 flower chain 0 handle 0x2
  eth_type ipv4
  src_ip 192.168.1.2
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 5 ref 1 bind 1
        used_hw_stats delayed

filter protocol ip pref 30 flower chain 0
filter protocol ip pref 30 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.3
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 6 ref 1 bind 1
        used_hw_stats delayed
```

**_Note_**: In DENT VM, this functionality is currently not supported, resulting in an error message like: "RTNETLINK answers: Operation not supported. We encountered an error while communicating with the kernel."

To observe statistics related to packets, bytes transmitted, or last time used, maintained on a per-rule basis, you can add the `-s` flag to the `tc` command:

```
tc -s filter show dev swp30 ingress
```

This command will display statistics for each filter rule on the `swp30` interface for incoming traffic.

```
filter protocol ip pref 10 flower chain 0
filter protocol ip pref 10 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.1
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 2 ref 1 bind 1 installed 492 sec
        Action statistics:
        Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
        backlog 0b 0p requeues 0
        used_hw_stats delayed

filter protocol ip pref 10 flower chain 0 handle 0x2
  eth_type ipv4
  src_ip 192.168.1.2
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 5 ref 1 bind 1 installed 357 sec
        Action statistics:
        Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
        backlog 0b 0p requeues 0
        used_hw_stats delayed

filter protocol ip pref 30 flower chain 0
filter protocol ip pref 30 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 192.168.1.3
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 6 ref 1 bind 1 installed 348 sec
        Action statistics:
        Sent 0 bytes 0 pkt (dropped 0, overlimits 0 requeues 0)
        backlog 0b 0p requeues 0
        used_hw_stats delayed
```

This output provides detailed statistics for each filter rule, including bytes transmitted, packets processed, packets dropped, and other relevant information.

Below are several examples demonstrating how to use `tc` with other supported ACL keys using the tc flower match:

To pass traffic with protocol 0x8FF:

```
tc filter add dev swp30 ingress pref 25 protocol 0x8FF flower skip_sw action pass
```

```
filter protocol bpq pref 25 flower chain 0
filter protocol bpq pref 25 flower chain 0 handle 0x1
  eth_type 08ff
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action pass
         random type none pass val 0
         index 7 ref 1 bind 1
        used_hw_stats delayed
```

To drop traffic with a specific source MAC address:

```
tc filter add dev swp30 ingress prio 24 flower skip_sw src_mac 00:11:22:33:44:88 action drop
```

```
filter protocol all pref 24 flower chain 0
filter protocol all pref 24 flower chain 0 handle 0x1
  src_mac 00:11:22:33:44:88
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 8 ref 1 bind 1
        used_hw_stats delayed
```

To drop all TCP traffic:

```
tc filter add dev swp30 ingress protocol ip flower skip_sw ip_proto tcp action drop
```

```
filter protocol ip pref 30 flower chain 0
filter protocol ip pref 30 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 9 ref 1 bind 1
        used_hw_stats delayed
```

To trap TCP traffic with a specific source port:

```
tc filter add dev swp30 ingress preference 43 protocol ip flower skip_sw ip_proto tcp src_port 39 action trap
```

```
filter protocol ip pref 43 flower chain 0
filter protocol ip pref 43 flower chain 0 handle 0x1
  eth_type ipv4
  ip_proto tcp
  src_port 39
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action trap
         random type none pass val 0
         index 10 ref 1 bind 1
        used_hw_stats delayed
```

To drop all traffic:

```
tc filter add dev swp30 ingress protocol all flower skip_sw action drop
```

```
filter protocol all pref 49151 flower chain 0
filter protocol all pref 49151 flower chain 0 handle 0x1
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 11 ref 1 bind 1
        used_hw_stats delayed
```

To drop IPv6 traffic with a specific source IP address:

```
tc filter add dev swp30 ingress protocol ipv6 flower skip_sw src_ip 1::2 action drop
```

```
filter protocol ipv6 pref 49150 flower chain 0
filter protocol ipv6 pref 49150 flower chain 0 handle 0x1
  eth_type ipv6
  src_ip 1::2
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 12 ref 1 bind 1
        used_hw_stats delayed
```

To view the configured filter rules:

```
tc filter show dev swp30 ingress
```

For IPv6 support in chain templates with matches on IPv6 addresses, filter rules are added implicitly for non-IPv6 traffic. To filter IPv6 traffic explicitly, additional rules must be added.

First, set up the clsact qdisc on device swp32:

```
tc qdisc add dev swp32 clsact
```

Then, add a filter rule to drop traffic with a specific source MAC address on ingress:

```
tc filter add dev swp32 ingress flower skip_sw src_mac 00:01:02:03:04:05 action drop
```

Next, add a filter rule specifically for IPv6 traffic with the same source MAC address:

```
tc filter add dev swp32 ingress protocol ipv6 flower skip_sw src_mac 00:01:02:03:04:05 action drop
```

To view the configured filter rules:

```
tc filter show dev swp32 ingress
```

```
filter protocol ipv6 pref 49151 flower chain 0
filter protocol ipv6 pref 49151 flower chain 0 handle 0x1
  src_mac 00:01:02:03:04:05
  eth_type ipv6
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 14 ref 1 bind 1
        used_hw_stats delayed

filter protocol all pref 49152 flower chain 0
filter protocol all pref 49152 flower chain 0 handle 0x1
  src_mac 00:01:02:03:04:05
  skip_sw
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 13 ref 1 bind 1
        used_hw_stats delayed
```

By executing the below commands, you are setting up a classless queuing discipline (`clsact`) on the network device `swp33`. Then, you define a chain template on the ingress direction of `swp33`, specifying that traffic matching the source MAC address `00:00:00:00:00:00` will be processed. Finally, you add a filter rule to drop traffic with the source MAC address `00:01:02:03:04:05` on ingress.

```
tc qdisc add dev swp33 clsact

tc chain add dev swp33 ingress chain 0 flower src_mac 00:00:00:00:00:00

tc filter add dev swp33 ingress flower skip_sw src_mac 00:01:02:03:04:05 action drop
```

This setup ensures that traffic with the specified source MAC address will be dropped upon arrival on device `swp33`.

**Delete ACL rules**

To remove a specific tc flower rule (ACL rule), it's deleted based on the delete criteria provided by the user.

To remove all rules from a specific qdisc, use the following command:

```
tc filter del dev swp30 root
```

After execution, you can verify the deletion by inspecting the remaining rules:

```
tc filter show dev swp30 ingress
```

If no output is displayed, it indicates that all rules have been successfully removed.

If an ACL is no longer needed on the DENT interface, you can use the following command to remove the qdisc along with all associated rules:

```
tc qdisc del dev swp33 parent ffff:
```

This command will effectively destroy the qdisc and remove all rules attached to it, ensuring that the ACL is completely removed from the interface.

**Hardware Statistics**

To create an ACL rule with delayed hardware statistics:

```
tc filter add dev swp31 ingress proto ip flower src_ip 1.1.1.0/2 action drop hw_stats delayed
```

```
src_ip 1.1.1.0/2 action drop hw_stats delayed
root@dentlab-infra1:~# tc filter show dev swp31 ingress
filter protocol ip pref 49152 flower chain 0
filter protocol ip pref 49152 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 1.1.1.0/2
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 2 ref 1 bind 1
        hw_stats delayed
        used_hw_stats delayed
```

To create a rule with disabled hardware statistics if no hardware counter is available:

```
tc filter add dev swp31 ingress proto ip flower src_ip 1.1.1.0/2 action drop hw_stats disabled
```

```
filter protocol ip pref 49151 flower chain 0
filter protocol ip pref 49151 flower chain 0 handle 0x1
  eth_type ipv4
  src_ip 1.1.1.0/2
  in_hw in_hw_count 1
        action order 1: gact action drop
         random type none pass val 0
         index 5 ref 1 bind 1
        hw_stats disabled
        used_hw_stats delayed
```

The outputs illustrate the creation of ACL rules with different hardware statistics settings. The first rule has delayed hardware statistics enabled, while the second rule has hardware statistics disabled.

<div style="border-top: 1px solid gray;"></div>

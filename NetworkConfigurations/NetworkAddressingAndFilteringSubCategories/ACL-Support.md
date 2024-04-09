---
title: Access Control Lists Support
grand_parent: Network Configuration
parent: Network Addressing and Filtering
nav_order: 3
layout: default
---

# Access Control Lists Support

ACL (Access Control List) support is a feature that allows administrators to control access to resources based on predefined rules. It's particularly important in managing network security and ensuring that only authorized users or systems can access specific resources.

**_Note_**: If you haven't already familiarized yourself with ACLs, you must refer to the [ACL guide](https://docs.dent.dev/NetworkConfigurations/NetworkAddressingAndFilteringSubCategories/ACL.html) first and then proceed here.

### Shared Block Support

Shared Block Support refers to the capability of ACLs to group multiple access control entries (ACEs) into a single block. This simplifies management by allowing administrators to apply common rules or permissions to multiple resources simultaneously.

For example, instead of individually specifying access rules for each file or folder on a file server, an administrator can create a shared block with rules applicable to all files or folders within a certain directory or category. This approach enhances efficiency and scalability in managing access control policies.

If you list the existing qdiscs, you see the block sharing information in the output:

```
tc qdisc
```

To share blocks, you need to request the share from the kernel at the qdisc creation stage:

```
tc qdisc add dev swp40 ingress_block 1 clsact
tc qdisc add dev swp41 ingress_block 1 clsact
```

```
qdisc pfifo_fast 0: dev swp40 root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc clsact ffff: dev swp40 parent ffff:fff1 ingress_block 1
qdisc pfifo_fast 0: dev swp41 root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc clsact ffff: dev swp41 parent ffff:fff1 ingress_block 1
```

Both ingress_block and egress_block can be added to the same qdisc with just one command:

```
tc qdisc add dev swp42 ingress_block 1 egress_block 2 clsact
```

```
qdisc pfifo_fast 0: dev swp42 root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc clsact ffff: dev swp42 parent ffff:fff1 ingress_block 1 egress_block 2
```

The number of qdiscs that can share the same block is not limited. Once the qdisc block is shared, you can no longer manipulate the filters using the dev handle. Instead, use the block index as a handle:

```
tc filter add block 1 protocol ip pref 20 flower dst_ip 192.168.0.0/16 action drop
```

In addition to clsact qdisc, block sharing is also supported for ingress qdisc:

```
tc qdisc add dev swp43 ingress_block 2 ingress
```

```
qdisc pfifo_fast 0: dev swp43 root refcnt 2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc ingress ffff: dev swp43 parent ffff:fff1 ingress_block 2 ----------------
```

### Multi-chain Support

Multichain Support extends the functionality of ACLs by enabling the creation of multiple access control chains within a single ACL policy. Each chain can have its own set of rules and priorities, providing finer granularity in controlling access to resources. This feature is particularly useful in complex network environments where different types of resources require distinct access control policies.

For instance, a network device may have separate chains for controlling access to user data, administrative functions, and network services. Multichain support allows administrators to tailor access control measures to the specific requirements of each resource category, enhancing security and flexibility.

To insert a filter into a specific chain, use the chain parameter of tc filter. Use the following command:

```
tc filter add dev <DEV> {ingress | egress} chain <CHAIN-INDEX> flower [MATCH(es)] action <ACTION>
```

A chain, if not created by the user, is created implicitly when a filter is added. Similarly, after the last filter is removed from the chain, it is destroyed. To explicitly create and destroy chains, use the following commands:

```
tc chain add dev <DEV> {ingress | egress} chain <CHAIN-INDEX>
tc chain del dev <DEV> {ingress | egress} chain <CHAIN-INDEX>
```

To show a list of tc chains, use the following command:

```
tc chain show dev <DEV> {ingress | egress}
```

**Examples**

To process rules in another chain, set the goto action of the filter in chain 0-

```
tc filter add dev swp44 ingress chain 0 flower skip_sw action goto chain 1
```

In order to view:

```
tc chain show dev swp44 ingress
```

```
chain parent ffff: chain 0
```

**_Note_**: Please make sure to create a qdisc before-

```
tc qdisc add dev swp44 ingress
```

Example to add and show:

```
tc chain add dev swp44 ingress chain 1

tc chain show dev swp44 ingress
```

```
chain parent ffff: chain 1
```

Example to delete and verify:

```
tc chain del dev swp44 ingress chain 1

tc chain show dev swp44 ingress
```

Empty output, as the chain has been deleted.

### Chain Template Support

Chain Templates support is a feature that allows administrators to define reusable templates for access control chains, streamlining the process of creating and managing complex ACL configurations. With Chain Templates, administrators can create standardized sets of access rules and parameters, which can then be applied across multiple access control chains, reducing configuration time and ensuring consistency.

For instance, in a network environment where different departments have varying access requirements, an administrator can create chain templates tailored to each department's needs, such as HR, Finance, and IT. These templates can then be easily applied to respective access control chains, simplifying management and ensuring uniform security policies across the organization.

Chain Template Support allows configuring templates in the chain, requiring explicit creation of a chain with a template. The configuration appears as follows:

```
tc chain add dev <DEV> {ingress | egress} chain <CHAIN-INDEX> flower <MATCH-TEMPLATE>
```

**Examples**

To create a chain with Source IP matched with Mask 16, use this command-

```
tc chain add dev swp44 ingress proto ip chain 1 flower src_ip 1.1.0.0/16
```

```
tc chain show dev swp44 ingress
```

```
chain parent ffff: flower chain 1
  eth_type ipv4
  src_ip 1.1.0.0/16
```

To create a chain in the ingress direction on device swp44, matching packets with specific IPv4 source and destination addresses-

```
tc chain add dev swp44 ingress chain 0 proto ip flower src_ip 1.1.1.1/32 dst_ip 2.0.0.0/8
```

```
chain parent ffff: flower chain 2
  eth_type ipv4
  dst_ip 2.0.0.0/8
  src_ip 1.1.1.1
```

To add a chain in the ingress direction on device swp44, targeting UDP packets with source ports falling within the range 150-200-

```
tc chain add dev swp44 ingress chain 3 proto ip flower ip_proto udp src_port 150-200
```

```
chain parent ffff: flower chain 3
  eth_type ipv4
  ip_proto udp
  src_port 150-200
```

**_Note_**: If a chain template creation fails, the command itself will not fail due to a kernel limitation. However, adding the rule into that chain will be rejected by the kernel, and an error is returned.

Example-

```
tc chain add dev swp44 ingress chain 1 proto ip flower ip_proto udp src_port 150-200
```

```
Error: Filter chain already exists.
We have an error talking to the kernel
```

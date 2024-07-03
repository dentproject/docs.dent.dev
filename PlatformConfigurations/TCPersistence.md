---
title: TC Persistence (Petunia)
parent: Platform Configuration
nav_order: 2
layout: default
---

# TC Persistence Petunia

## Introduction

The package Petunia
is a set of scripts to aid in writing
TC-flower ACLS for DENT/switchdev devices. In this guide, we will cover how to install Petunia
and use the commands for these scripts.

## Petunia Installation

Before cloning the remote repository
do not forget to use `$ apt-get update` to
fetch the latest version of your package lists.
Follow this with the command `$ apt-get upgrade` to first
review the changes in the latest versions
and then replace the old packages by installing the new ones.

To install Petunia:

1. Clone the petunia repository from Github onto your device
   and enter the directory

```
$ git clone https://github.com/dentproject/petunia.git
```

2. Next, enter the cloned directory and install the manager for Python packages and the necessary
   Python3 libraries

```
$ cd petunia
$ apt install python3-pip python3-all -y
```

3. Install the required supplemental utilities

```
$ pip3 install stdeb
$ pip3 install pyroute2
```

4. Setup the Debian package, then install it

```
$ python3 setup.py --command-packages=stdeb.command bdist_deb
$ cd deb_dist
$ dpkg -i python3-petunia_1.0-1_all.deb
```

---

## Commands

The following is a list of the commands that are added with Petunia:

- **tc-flower-show** - Display the tc rules in a tabular format.

- **iptables-unroll** - Compile JUMP/GOTO statements in an iptables-save ruleset
  file into a compatible format. The output format is "mostly" compatible with
  IPTABLES, but uses chain targets that IPTABLES does not directly support.

- **iptables-scoreboard** - Consume the output of iptables-unroll, attempting to compress the rules by
  grouping them by ingress interface. Adjacent rules that transitively span all
  interfaces (e.g. swp+) or span all trunk ports for a given VLAN ID can be simplified/merged.

- **iptables-slice** - Compute the subset of interfaces to operate on from 'allInterfaces'
  to interface specific chains.
  Assume here that the original chain is already unrolled.
  If interfaces are specified in 'onlyInterfaces,' this restricts
  the interfaces we support to just those.

- **iptables-unslice** - Combine iptables rules that have been
  previously sliced into interface-specific chains back into a more general set of rules.

- **tc-flower-load** - Do the actual conversion from iptables rules to tc rules. Refer to the synopsis for the exact
  field conversion.

## Synopsis

- **tc-flower-show** [OPTIONS][-b BLOCK] [-i INTERVAL] [INTERFACE...]

```
OPTIONS :=
-v | --verbose Verbose logging
-i | --interval display the tc rules every INTERVAL seconds
-b | --block block to display
-n | --non-zero Display only tc rules that are hit
BLOCK:= block to display
INTERVAL:= display every INTERVAL seconds
INTERFACE:= list corresponds to all (front-panel) interfaces
```

- **iptables-unroll** iptables-unroll [OPTIONS] SRC DST [CHAIN]

```
OPTIONS :=
-- multi-chain Allow multiple chain specifiers (separated by commas)
-- multi-interface Allow multiple interface specifiers (separated by commas)
-- override-interface Allow interface overrides in chain targets
   (default is to raise an error)
-- extended Support IPTABLES target extensions
CHAIN:= defaults to FORWARD
INTERFACE:= list corresponds to all (front-panel) interfaces
SRC:= Example use case is the output of 'iptables-save'
DST:= Name and location of where to save the file
```

- **iptables-scoreboard** [OPTIONS] SRC DST [CHAIN] [INTERFACE...]

```
OPTIONS :=
-v | -- verbose Verbose logging
-q | --quiert Quiet logging
--strict Do not generate rules for unspecified (wildcard) interfaces
-h | --help|-? Print this help
CHAIN := defaults to FORWARD
INTERFACE:= list corresponds to all (front-panel) interfaces
    Without --strict, we can generate rules that affect un-specified interfaces
    With --strict, we specify verbose rules not to affect un-specified interfaces,
    at the cost of rule length
SRC:= Example use case is the output of 'iptables-unroll'
DST:= Name and location of where to save the file
```

- **iptables-slice** SRC DST [CHAIN] [INTERFACE ...]

```
CHAIN := defaults to FORWARD
INTERFACE:= list corresponds to all (front-panel) interfaces
SRC:= Example use case is the output of 'iptables-save'
DST:= Name and location of where to save the file
```

- **iptables-unslice** [OPTIONS] SRC DST [CHAIN] [INTERFACE ...]

```
OPTIONS :=
-4|-6          Select IPv4 vs IPv6
--merge [STRATEGY]
--no-merge
Merge strategy for shared-block loading

Merge STRATEGY is one of

    exact        Merge identical chains
    suffix       Merge chains that have common suffixes
                 (implies exact)
    prefix       Merge chains that have common prefixes
                 (implies exact)
    all|default  Default strategy (exact+suffix+prefix)

CHAIN := defaults to FORWARD
INTERFACE:= list corresponds to all (front-panel) interfaces
SRC:= Example use case is the output of 'iptables-slice'
DST:= Name and location of where to save the file
```

- **tc-flower-load** [OPTIONS] SRC [CHAIN] [INTERFACE...]

```
OPTIONS :=
    -4| -6 Select IPv4 or IPV6
    -q| --quiet Suppress warning messages
    -v|--verbose Show debug messages (including command traces)
    --non-atomic Quick-load rules set with non-atomic updates
    --drop Quick-load rules sets with a default-drop policy (atomic but
        otherwise destructive)
    --offload
    --no-offload Force/disallow hardware offload
    --shared-block
    --no-shared-block Choose a filter load strategy (one chain per interface
        vs one shared block for all interfaces)
    --scoreboard Output is from 'iptables-scoreboard' instead of 'iptables-unslice'
    --log-ignore Ignore LOG targets
    --police-accept Accept traffic for POLICE targets (WARNING!)
    --addrtype-fail Treat addrtype matches as pass (always match) or fail (never match)
    --setclass-ignore Ignore the SETCLASS target
    --multi-match Support coma-separated matches for port, IP address
    --port-unroll N Unroll statements of the form --sport M:M+N into individual
        port matches
    --hack-vlan-arp Always vlan-encapsulated ARPs (needed if the existing rules
        drop all TCP/UDP, which TC inadvertently interprets as "all IP, including ARP")
    -- drop-mode trap
    -- drop-mode trap, RATE
    -- drop-mode trap, RATE, BURST
        Configure handling for dropped packets
            -'trap' --> trap dropped packets to the CPU for logging
            -'trap,RATE,BURST' --> trap/log packets below a given RATE,
                                    allow for bursts, drop the rest
        Specifying 'trap' without RATE,BURST is not RECOMMENDED!
    --profile Enable call profiling
    --no-batch      Do not batch TC commands
    --reject-drop   Drop packets with a REJECT target (otherwise report an error)
    --continue-surpress Suppress no-op rules (TC 'continue' actions)
            Enable this if your switchdev implementation does not offload the 'continue' action
    -h--help|-? Print this help
CHAIN := defaults to FORWARD
INTERFACE:= list corresponds to all (front-panel) interfaces
SRC:= should be the output of 'iptables-slice' (for -no-shared-block) or
            'iptables-unslice' (for -shared-block)
```

## Example Usages

- **tc-flower-show**

To test `tc-flower-show,` let's first configure a qdisc on a shared block 1
with a tc-flower filter.

Ex.

```
$ sudo tc qdisc add dev enp0s3 ingress_block 1 clsact
$ sudo tc filter add block 1 ingress flower action drop
```

Running the `tc-flower-show` command will now result in:

```
Pref 49152 Chain 0 protocol all Key [indev any] Action [drop Pkt 8 Bytes 368 HW Pkt 0 Bytes 0]
```

- **iptables-unroll**

To test iptables-unroll, create and save an iptables rule set.

Ex.

```
$ sudo iptables -A FORWARD -i enp0s3 -o lo -j DROP
$ iptables-save -t filter > /etc/iptables.rules
```

Next, unroll the save iptables filters:

```
$ iptables-unroll --multi-interface --extended /etc/iptables.rules /etc/iptables-unrolled.rules FORWARD
```

The resulting output after `iptables-unroll` will be:

```
INFO:iptables-unroll: reading iptables rules from /etc/iptables.rules
INFO:iptables-unroll: unrolling 1 rules in chain FORWARD
INFO:iptables-unroll: emitting 1 rules to /etc/iptables-unrolled.rules
```

- **iptables-scoreboard**

To test iptables scoreboard first create then save an iptables rule set.

Ex.

```
$ sudo iptables -A FORWARD -i enp0s3 -o lo -j DROP
$ iptables-save -t filter > /etc/iptables.rules
```

Next, unroll the save iptables filters:

```
$ iptables-unroll --multi-interface --extended /etc/iptables.rules /etc/iptables-unrolled.rules FORWARD
```

Finally, with the unrolled rules execute the iptables-scoreboard command:

```
$ iptables-scoreboard /etc/iptables-unrolled.rules /etc/iptables-scoreboard.rules FORWARD enp0s3
```

The resulting output after `iptables-scoreboard` will be:

```
INFO:iptables-scoreboard:reading iptables rules from /etc/iptables-unrolled.rules
INFO:iptables-scoreboard:scoreboarding 1 rules in chain FORWARD
INFO:iptables-scoreboard:emitting 1 rules to /etc/iptables-scoreboard.rules
```

- **iptables-slice**

To test iptables-slice, first create and then save an iptables rule set.

Ex.

```
$ sudo iptables -A FORWARD -i enp0s3 -o lo -j DROP
$ iptables-save -t filter > /etc/iptables.rules
```

Next, unroll the saved iptables filters:

```
$ iptables-unroll --multi-interface --extended /etc/iptables.rules /etc/iptables-unrolled.rules FORWARD
```

Then, slice the unrolled filters.

```
$ iptables-slice /etc/iptables-unrolled.rules /etc/iptables-sliced.rules FORWARD enp0s3
```

The following result after `iptables-slice` is:

```
DEBUG:iptables-slice.links:getting link info from kernel
DEBUG:iptables-slice.links:found 1 physical link(s), 1 other link(s)
DEBUG:iptables-slice.links:getting vlan info from kernel
DEBUG:iptables-slice.links:found 0 vlan links
DEBUG:iptables-slice.links:found 0 bridge(s), 0 vlan(s) while probing vlans
INFO:iptables-slice:reading iptables rules from /etc/iptables-unrolled.rules
INFO:iptables-slice:slicing 1 rules in chain FORWARD
DEBUG:iptables-slice.slice.links:getting link info from kernel
DEBUG:iptables-slice.slice.links:found 1 physical link(s), 1 other link(s)
DEBUG:iptables-slice.slice.links:getting vlan info from kernel
DEBUG:iptables-slice.slice.links:found 0 vlan links
DEBUG:iptables-slice.slice.links:found 0 bridge(s), 0 vlan(s) while probing vlans
INFO:iptables-slice:emitting 1 rules to /etc/iptables-sliced.rules
```

- **iptables-unslice**

  To test iptables-slice, first create and then save an iptables rule set.

Ex.

```
$ sudo iptables -A FORWARD -i enp0s3 -o lo -j DROP
$ iptables-save -t filter > /etc/iptables.rules
```

Next, slice the saved iptables filters:

```
$ iptables-slice /etc/iptables.rules /etc/iptables-sliced.rules FORWARD enp0s3
```

Finally, unslice the sliced rules:

```
$ iptables-unslice -4 -–merge /etc/iptables-sliced.rules /etc/iptables-unsliced.rules
```

The following result after `iptables-unslice` is:

```
DEBUG:iptables-unslice.links:getting link info from kernel
DEBUG:iptables-unslice.links:found 1 physical link(s), 1 other link(s)
DEBUG:iptables-unslice.links:getting vlan info from kernel
DEBUG:iptables-unslice.links:found 0 vlan links
DEBUG:iptables-unslice.links:found 0 bridge(s), 0 vlan(s) while probing vlans
INFO:iptables-unslice:reading iptables rules from /etc/iptables-sliced.rules
INFO:iptables-unslice:unslicing 1 rules in chain FORWARD
DEBUG:iptables-unslice.unslice.links:getting link info from kernel
DEBUG:iptables-unslice.unslice.links:found 1 physical link(s), 1 other link(s)
DEBUG:iptables-unslice.unslice.links:getting vlan info from kernel
DEBUG:iptables-unslice.unslice.links:found 0 vlan links
DEBUG:iptables-unslice.unslice.links:found 0 bridge(s), 0 vlan(s) while probing vlans
DEBUG:iptables-unslice.unslice.slice.links:getting link info from kernel
DEBUG:iptables-unslice.unslice.slice.links:found 1 physical link(s), 1 other link(s)
DEBUG:iptables-unslice.unslice.slice.links:getting vlan info from kernel
DEBUG:iptables-unslice.unslice.slice.links:found 0 vlan links
DEBUG:iptables-unslice.unslice.slice.links:found 0 bridge(s), 0 vlan(s) while probing vlans
INFO:iptables-unslice.unslice.slice:targeted interfaces are enp0s3
INFO:iptables-unslice.unslice.slice:source table has 1 rules
INFO:iptables-unslice.unslice.slice:trying merge strategy simpleMerge
INFO:iptables-unslice.unslice.slice:merged table has 2 rules
INFO:iptables-unslice.unslice.slice:trying merge strategy simpleSuffixMerge
DEBUG:iptables-unslice.unslice.slice:computed 1 unique hashes
DEBUG:iptables-unslice.unslice.slice:examining chain enp0s3 (1 rules)
INFO:iptables-unslice.unslice.slice:merged table has 2 rules
INFO:iptables-unslice.unslice.slice:trying merge strategy <lambda>
DEBUG:iptables-unslice.unslice.slice:computed 1 unique hashes
DEBUG:iptables-unslice.unslice.slice:examining chain enp0s3 (1 rules)
WARNING:iptables-unslice.unslice.slice:no valid candidates to merge for enp0s3
INFO:iptables-unslice.unslice.slice:merged table has 2 rules
INFO:iptables-unslice:emitting 3 rules to /etc/iptables-unsliced.rules
```

- **iptables-load**

An example use of iptables-load, first create and then save an iptables rule set.

Ex.

```
$ sudo iptables -A FORWARD -i enp0s3 -o lo -j DROP
$ iptables-save -t filter > /etc/iptables.rules
```

Next, slice the save iptables filters:

```
$ iptables-slice /etc/iptables.rules /etc/iptables-sliced.rules FORWARD enp0s3
```

Unslice the sliced rules:

```
$ iptables-unslice -4 --merge /etc/iptables-sliced.rules /etc/iptables-unsliced.rules
```

Finally, convert iptables rules to tc rules using the iptables-unsliced file

```
$ tc-flower-load -v --no-offload --port-unroll 5 --shared-block --hack-vlan-arp --log-ignore --drop-mode trap,1Kbps,100 --continue-suppress  /etc/iptables-unsliced.rules
```

The following result after `iptables-load` is:

```
DEBUG:tc-flower-load.links:getting link info from kernel
DEBUG:tc-flower-load.links:found 1 physical link(s), 1 other link(s)
DEBUG:tc-flower-load.links:getting vlan info from kernel
DEBUG:tc-flower-load.links:found 0 vlan links
DEBUG:tc-flower-load.links:found 0 bridge(s), 0 vlan(s) while probing vlans
INFO:tc-flower-load:reading processed iptables rules from /etc/iptables-unsliced.rules
DEBUG:tc-flower-load.load.links:getting link info from kernel
DEBUG:tc-flower-load.load.links:found 1 physical link(s), 1 other link(s)
DEBUG:tc-flower-load.load.links:getting vlan info from kernel
DEBUG:tc-flower-load.load.links:found 0 vlan links
DEBUG:tc-flower-load.load.links:found 0 bridge(s), 0 vlan(s) while probing vlans
DEBUG:tc-flower-load.load:+ tc -json qdisc show
DEBUG:tc-flower-load.load:+ exit 0
INFO:tc-flower-load.load:adding ingress qdisc to enp0s3 (via shared block 1)
DEBUG:tc-flower-load.load:+ tc qdisc add dev enp0s3 ingress_block 1 ingress
DEBUG:tc-flower-load.load:+ exit 0
DEBUG:tc-flower-load.load:+ tc -json filter show block 1 ingress
DEBUG:tc-flower-load.load:+ exit 0
DEBUG:tc-flower-load.load:found 0 filters attached to block 1
INFO:tc-flower-load.load:generating rules for block 1
WARNING:SkipClause:invalid target args with 'SKIP': ['--skip-rules', '0']: no-op
WARNING:SkipClause:invalid target args with 'SKIP': ['--skip-rules', '0']: no-op
DEBUG:tc-flower-load.load.translate:rule index 0: target IPTABLES line 2 --> jump 0
DEBUG:tc-flower-load.load.translate:rule index 2: target IPTABLES line 4 --> jump 0
INFO:tc-flower-load.load:loading 4 rules for block 1 at 32768
INFO:tc-flower-load.load.tc:invoking 4 TC commands in batch mode...
DEBUG:tc-flower-load.load.tc:+ tc -batch /tmp/tc-flower-1ss_qcza.in
DEBUG:tc-flower-load.load.tc:+ exit 0
```

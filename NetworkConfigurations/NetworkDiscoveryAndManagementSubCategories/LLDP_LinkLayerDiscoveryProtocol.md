---
title: LLDP (Link Layer Discovery Protocol)
grand_parent: Network Configuration
parent: Network Discovery and Management
nav_order: 2
layout: default
---

# LLDP (Link Layer Discovery Protocol)

## Introduction

In this guide, we will quickly explain what the
Link Layer Discovery Protocol is
and give an example implementing it.

The Link Layer Discovery Protocol or LLDP is a network protocol that operates
on the Layer 2 level of devices. LLDP is an IEEE standard that allows devices to gather
information about neighboring devices.
The defined set of attributes that LLDP uses to uncover information about
neighbor devices are referred to as TLVs.

LLDPDU is a multicast frame or data unit that is sent out
containing the information that is shared between devices.

In this example, we will be using the `lldpd` utility. `lldpd` is an
IEEE 802.1AB implementation of LLDP with support for custom TLVs.

The following are the mandatory TLVs sent in an
LLDPDU:

| TLV Name      |                          Description                          |
| ------------- | :-----------------------------------------------------------: |
| End of LLDPDU |                    End of TLV Information                     |
| Chassis ID    |                     Identifies the Device                     |
| Port ID       | By default, lldpd will use the MAC address as port identifier |
| Time to Live  |    How long received informaiton<br/> should remain valid     |

Optional TLVs in 802.1AB include:

| TLV Name            |                  Description                   |
| ------------------- | :--------------------------------------------: |
| Port Description    |            Displays Interface Name             |
| System Name         |               Displays Hostname                |
| System Description  |         Displays Version of the System         |
| System Capabilities |         Bridge or Routing Capabilities         |
| Management Address  | Specifies the management address of the device |

TLVs included in IEEE 802.1/IEEE 802.3 Organizationally Specific TLVs are not
supported by the agent
but can be configured statically via custom TLVs. Some examples include information such as:

| TLV Name          |                      Description                       |
| ----------------- | :----------------------------------------------------: |
| Vlan ID and Names |            The associated VLAN ID and Names            |
| MDI and POE       | Details for if the device supports Power over Ethernet |
| Link Aggregation  |        Information on links that are aggregated        |

## Enabling Link Layer Discovery Protocol

This section describes how to configure LLDP using an LLDPd agent.

If the `lldpd` utility is not already installed:

_Do not forget to use `$ apt-get update` to
fetch the latest version of your package lists.
Follow this with the command `$ apt-get upgrade` to first
review the changes in the latest versions
and then replace the old packages by installing the new ones._

To install the `lldpd` utility run the following: `$ apt-get install lldpd`

This utility has support for the following:

- LLDP
- CDP (Cisco)
- EDP (Extreme)
- SONMP (Nortel)
- FDP (Foundry)

Once installed on more than one device, connected devices may run the following
to view information about individuals they are connected to:

```
$ lldpcli show neighbors
```

Additional informaiton can be queried by adding `details` to the command above

Ex

```
$ lldpcli show neighbors details
```

To update its information and send a new LLDPDU on all interfaces, run the following:

```
$ lldpcli update
```

To show the current configuration, use:

```
$ lldpcli show configuration
```

## Queries, Configurable, and Custom TLVs

**NOTE: By default, the LLDPd agent enables LLDP on all available physical interfaces.**

The following includes queryable information
and how to limit LLDP to specific ports or interfaces.

### Query TLV Information about a Connected Neighbor

To view information about individuals, a device is connected to run the following:

```
$ lldpcli show neighbors
```

### Query LLDP statistics on a port

To query LLDP statistics on a port, run the following:

```
lldpcli show statistics ports ${Interface Name}
```

Ex.

```
lldpcli show statistics ports enp0s4
```

### Limit the LLDPd agent to a specific port

To limit the LLDPd agent to a specific port, use the following command:

```
$ lldpcli configure system interface pattern ${Interface name}
```

This command specifies which interface to listen to and send LLDPDU to. Without
this option, lldpd will use all available physical interfaces.

Ex.

```
$ lldpcli configure system interface pattern enp0s4
```

### Enable/disable LLDP on a specific interface

To enable/disable ingress or egress LLDPDU traffic
on a specific port, use the following command:

```
lldpcli configure ports ${Interface Name} lldp status ${OPTION}
```

The following options are available:

`rx-and-tx` - Rx and Tx means devices can receive and transmit LLDP frames.

`rx-only` - In rx-only mode, they won't emit any frames.

`tx-only` - In tx-only mode, they won't receive any frames.

`disabled` - In disabled mode, no frame will be sent, and any incoming frames will be discarded.

Ex.

```
lldpcli configure ports enp0s4 lldp status rx-and-tx
```

### Creating Custom TLVs

To create a custom TLV, use the following outline:

```
$ lldpcli configure ${[ports ethX [,…]]} lldp custom-tlv ${[add | replace]} oui ${oui} subtype ${subtype} ${[oui-info content]}
```

Both the oui and oui-info content should be a comma-separated list of
bytes in hex format. The oui must be exactly 3 bytes long.
Unless `replace` is specified the default action will be to add the newly created
custom TLV. If `replace` is specified, all TLVs
with the same oui and subtype will be replaced by the newly defined custom TLV.

Ex.

```
$ lldpcli configure lldp custom-tlv oui 00,80,c2 subtype 1 oui-info 56,78,9,0,90,78,54
```

For more information on configuring devices and custom TLV's
with the LLDPd agent, visit the following: [LLDPD Man Page](https://lldpd.github.io/usage.html)

---

## Example Configuration

Consider the following topology:

![LLDP_Topology](../../Images/ImagesForNetworkConfiguration/LLDP_Topology.png)

Imagine DENT3.2-1 (DENT1) wanted to confirm whether DENT3.2-2 (DENT2)
is configured for full or half duplex.

If the lldpd utility is not already installed, run the following on both devices:

_Do not forget to use `$ apt-get update` to
fetch the latest version of your package lists.
Follow this with the command `$ apt-get upgrade` to first
review the changes in the latest versions
and then replace the old packages by installing the new ones._

To install the lldpd utility, run the following: `$ apt-get install lldpd`

Once installed by default, LLDP will automatically be enabled.

Ensure the interfaces between the two devices are up by running
the following on both switches: ` $ip link set enp0s4 up`

Connected devices will then be able to view information regarding
connected devices with the following command:

```
$ lldpcli show neighbors details
```

See below:

```
root@DENT1:~# lldpcli show neighbors details
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
Interface:    ma1, via: LLDP, RID: 4, Time: 0 day, 03:34:28
  Chassis:
    ChassisID:    mac 0c:49:40:99:00:00
    SysName:      DENT2
    SysDescr:     Debian GNU/Linux 9 (stretch) Linux 5.6.16-OpenNetworkLinux #1 SMP Thu Jun 22 22:46:37 UTC 2023 x86_64
    TTL:          120
    MgmtIP:       172.24.206.122
    MgmtIP:       fe80::e49:40ff:fe99:0
    Capability:   Bridge, off
    Capability:   Router, off
    Capability:   WLAN, off
    Capability:   Station, on
  Port:
    PortID:       mac 0c:49:40:99:00:00
    PortDescr:    ma1
    PMD autoneg:  supported: yes, enabled: yes
      Adv:          10Base-T, HD: yes, FD: yes
      Adv:          100Base-TX, HD: yes, FD: yes
      Adv:          1000Base-T, HD: no, FD: yes
      MAU oper type: 1000BaseTFD - Four-pair Category 5 UTP, full duplex mode
-------------------------------------------------------------------------------
root@DENT1:~#
```

Above is information about the DENT2
device queried from the DENT1 device.
DENT2 is shown to be operating at full duplex.
Other relevant information about the device is also shared.

**NOTE: The outputs above were tested on a Virtual Machine**

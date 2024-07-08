---
title: Power over Ethernet Management
parent: Platform Configuration
nav_order: 4
layout: default
---

# Power over Ethernet Management

Power over Ethernet (PoE) technology allows network cables to carry electrical power, enabling devices to operate without a separate power supply.

The PoE agent is a software package on DentOS that assists users in managing the PoE chip on their board. This agent ensures that the system can automatically restore user configurations from cold boot, warm boot, or even after a system crash.

This document has three parts: the Installation Guide, the User Guide, which provides guidelines for command operations, and Troubleshooting, which helps handle runtime problems.

If `poecli` commands are not working or are not already installed on your device, follow the instructions below:

## Installation

To install PoE agent, follow these steps:

### 1. Update the package list:

```
sudo apt-get update
```

### 2. Install necessary packages:

```
sudo apt-get install python3-stdeb fakeroot python-all dh-python
```

### 3. Clone the PoE project repository:

```
git clone https://github.com/dentproject/poed.git
cd poed
```

### 4. Build the Debian package:

```
bash build-deb.sh
```

5. **Start the PoE agent:**

```
    systemctl restart poed
```

> **Note:** Refer to the troubleshooting section at the end of this document if the agent does not start properly.

## User Guide

This guide provides a comprehensive introduction to the `poecli` commands, offering detailed explanations and usage instructions for each command.

### 1. Subcommands Help:

```
poecli -h

usage: poecli.py [-h] Commands ...

positional arguments:
  Commands            Descriptions
    show              Show PoE information
    set               Set PoE ports
    savechip          Save current user values to non-volatile memory to become
                      defaults after resets. Use 'poecli restore_poe_system' to
                      revert to factory defaults. Persistent config changes
                      require 'poecli cfg --save'.
    cfg               Manipulate poe agent config files.
    restore_poe_system
                      Restore POE chip to factory default values included in
                      firmware release. Ports will temporarily shut down. After
                      restoration, initialize port settings for the platform.
                      Persistent config changes require 'poecli cfg --save'.

optional arguments:
  -h, --help          show this help message and exit
```

### 2. Configuration File Manipulation (Requires PoE agent started):

**2.1. Display config subcommand help:**

```
poecli cfg -h

usage: poecli.py cfg [-h] [-s] [-l] [-c <val>]

optional arguments:
  -h, --help          show this help message and exit
  -s, --save          Save current runtime settings to persistent file.
  -l, --load          Load settings from persistent file.
  -c <val>, --config <val>
                      Specify file path for save/load operation instead of the
                      default persistent config. Example: poecli cfg -s -c
                      [Config Path]
```

**2.2. Create Persistent Config from current chip state (/etc/poe_agent/poe_perm_cfg.json):**

```
poecli cfg -s

cfg_action: poecli_cfg,save
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:02:15 localhost poed.py[1493]: INFO: Save chip state to /etc/poe_agent/poe_perm_cfg.json
Jul 09 00:02:15 localhost poed.py[1493]: INFO: Action: poecli_cfg,save
```

**2.3. Load Persistent Config (/etc/poe_agent/poe_perm_cfg.json):**

```
poecli cfg -l

cfg_action: poecli_cfg,load
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:03:16 localhost poed.py[1493]: INFO: Load config from /etc/poe_agent/poe_perm_cfg.json
Jul 09 00:03:16 localhost poed.py[1493]: INFO: Action: poecli_cfg,load
```

**2.4. Create Persistent Config from current chip state to a specified file (absolute path):**

```
poecli cfg -s -c /root/test.json

cfg_action: poecli_cfg,save,/root/test.json
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:04:55 localhost poed.py[1493]: INFO: Save chip state to /root/test.json
Jul 09 00:04:55 localhost poed.py[1493]: INFO: Action: poecli_cfg,save,/root/test.json
```

**2.5. Load Config file from a specified path (absolute path):**

```
poecli cfg -l -c /root/test.json

cfg_action: poecli_cfg,load,/root/test.json
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:05:25 localhost poed.py[1493]: INFO: Load config from /root/test.json
Jul 09 00:05:25 localhost poed.py[1493]: INFO: Action: poecli_cfg,load,/root/test.json
```

### 3. Set Port Parameters:

**3.1. Display set subcommand help:**

```
poecli set -h

usage: poecli.py set [-h] -p <val> [-e <val>] [-l <val>] [-o <val>]

optional arguments:
  -h, --help            show this help message and exit
  -p <val>, --ports <val>
                        Logic ports
                        Example: 1,3-5,45-48
  -e <val>, --enable <val>
                        Port Enable/Disable
                        disable = 0, enable = 1
  -l <val>, --level <val>
                        Port Priority Level
                        crit = 1, high = 2, low = 3
  -o <val>, --powerLimit <val>
                        Port Power Limit
                        range: 0x0 (mW) - 0xffff (mW)
                        This field will be ignored if val sets to 0xffff

```

**3.2. Enable port:**

```
poecli set -p 1 -e 1
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:06:42 localhost poed.py[1493]: INFO: Set port 1 enable
Jul 09 00:06:42 localhost poed.py[1493]: INFO: Action: poecli_set,enable,1
```

**3.3. Disable port in range:**

```
poecli set -p 1-48 -e 0
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:07:20 localhost poed.py[1493]: INFO: Set ports 1-48 disable
Jul 09 00:07:20 localhost poed.py[1493]: INFO: Action: poecli_set,disable,1-48
```

**3.4. Enable ports (some skipped) with power Limit and priority:**

```
poecli set -p 1,2,5,7 -e 1 -o 0x1fe0 -l 1
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:08:41 localhost poed.py[1493]: INFO: Set ports 1,2,5,7 enable with power limit 0x1fe0 and priority level 1
Jul 09 00:08:41 localhost poed.py[1493]: INFO: Action: poecli_set,enable,1,2,5,7,powerLimit,0x1fe0,level,1
```

### 4. Show Information Subcommand:

**4.1. Display Show information help:**

```
poecli show -h

usage: poecli.py show [-h] [-d] [-j] [-p <val> | -s | -m | -a | -v]

optional arguments:
  -h, --help            show this help message and exit
  -d, --debug           Show more Information for debugging
  -j, --json            Display information in JSON format
  -p <val>, --ports <val>
                        Show PoE Ports Information
                        Example: 1,3-5,45-48
  -s, --system          Show PoE System Information
  -m, --mask            Show Individual mask registers
  -a, --all             Show port, system, and individual masks Information
  -v, --version         Show PoE versions
```

**4.2. Show port info:**

```
poecli show -p 1,2,5,7

Port  Status          En/Dis  Priority  Protocol       Class  PWR Consump  PWR Limit  Voltage  Current
----  --------------  ------  --------  --------------  -----  -----------  ---------  -------  -------
1     Port On (0x01)  enable  crit      IEEE802.3AF/AT  4      500 (mW)     8100 (mW)  54.3 (V)  8 (mA)
2     Port Off (0x1B) enable  crit      IEEE802.3AF/AT  0      0 (mW)       8100 (mW)  0.0 (V)   0 (mA)
5     Port Off (0x1B) enable  crit      IEEE802.3AF/AT  0      0 (mW)       8100 (mW)  0.0 (V)   0 (mA)
7     Port Off (0x1B) enable  crit      IEEE802.3AF/AT  0      0 (mW)       8100 (mW)  0.0 (V)   0 (mA)
```

**4.3. Show system info:**

```
poecli show -s

==============================
PoE System Information
==============================
Total PoE Ports   : 48

Total Power       : 1500.0 W
Power Consumption : 0.0 W
Power Available   : 1500.0 W

Power Bank #      : 15
Power Sources     : PSU1, PSU2
```

**4.4. Show individual mask:**

```
poecli show -m

==================
Individual Masks
==================
0x00: 1
.................(skip logs)
0x53: 0
```

**4.5. Show all information:**

```
poecli show -a

Port Info:
Port 1:
    Status: Port On (0x01)
    Enable/Disable: enable
    Priority: crit
    Protocol: IEEE802.3AF/AT
    Class: 4
    Power Consumption: 500 mW
    Power Limit: 8100 mW
    Voltage: 54.3 V
    Current: 8 mA

Port 2:
    Status: Port Off (0x1B)
    Enable/Disable: enable
    Priority: crit
    Protocol: IEEE802.3AF/AT
    Class: 0
    Power Consumption: 0 mW
    Power Limit: 8100 mW
    Voltage: 0.0 V
    Current: 0 mA

Port 5:
    Status: Port Off (0x1B)
    Enable/Disable: enable
    Priority: crit
    Protocol: IEEE802.3AF/AT
    Class: 0
    Power Consumption: 0 mW
    Power Limit: 8100 mW
    Voltage: 0.0 V
    Current: 0 mA

Port 7:
    Status: Port Off (0x1B)
    Enable/Disable: enable
    Priority: crit
    Protocol: IEEE802.3AF/AT
    Class: 0
    Power Consumption: 0 mW
    Power Limit: 8100 mW
    Voltage: 0.0 V
    Current: 0 mA

PoE System Information:
Total PoE Ports   : 48

Total Power       : 1500.0 W
Power Consumption : 0.0 W
Power Available   : 1500.0 W

Power Bank #      : 15
Power Sources     : PSU1, PSU2


Individual Masks:
0x00: 1
.................(skip logs)
0x53: 0
```

**4.6. Show info in JSON format (for some other applications):**

```
poecli show -p 1 -j

{
    "PORT_INFORMATION": [
        {
            "port_id": 1,
            "enDis": "enable",
            "priority": "crit",
            "power_limit": 8100,
            "status": "Port On (0x01)",
            "latch": 0,
            "protocol": "IEEE802.3AF/AT",
            "enable_4pair": 0,
            "class": "4",
            "power_consump": 500,
            "voltage": 54.3,
            "current": 10
        }
    ]
}
```

### 5. Store Current Chip State to its NV-MEMORY (savechip):

```
poecli savechip
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:10:26 localhost poed.py[1493]: INFO: Receive a set event from poecli!
Jul 09 00:10:26 localhost poed.py[1493]: INFO: Save chip state to NV-MEMORY
```

### 6. Restore POE Chip to Factory Default and Apply Platform Default (restore_poe_system):

```
poecli restore_poe_system
```

Select 2-Pair mode. Port map mismatch, run program global matrix. Program active matrix, all ports will shut down a while. Program active matrix completed, save platform settings to chip. Success to restore factory default and take platform PoE settings!

(Suggest restart poe agent or reboot):

```
systemctl restart poed
```

Check agent log:

```
journalctl -u poed -n 10

Jul 09 00:10:33 localhost poed[25224]: SystemExit: 0
Jul 09 00:10:33 localhost systemd[1]: Stopped DentOS POE Agent.
Jul 09 00:10:33 localhost systemd[1]: Started DentOS POE Agent.
Jul 09 00:10:33 localhost poed.py[25248]: INFO: Configure PoE ports from "/run/poe_runtime_cfg.json"
Jul 09 00:10:33 localhost poed[25242]: Select 2-Pair mode
Jul 09 00:10:34 localhost poed[25242]: Port map match, skip program global matrix
Jul 09 00:10:35 localhost poed.py[25248]: INFO: init_poe all_result: 0
Jul 09 00:10:35 localhost poed.py[25248]: INFO: Success to initialize platform PoE settings!
Jul 09 00:10:41 localhost poed.py[25248]: INFO: Success to restore port configurations from "/run/poe_runtime_cfg.json".
Jul 09 00:10:41 localhost poed.py[25248]: INFO: Start autosave thread
```

## Troubleshooting

### POE Agent Cannot Start (Config File Issue):

Check agent log:

```
journalctl -u poed -n 15

Jul 09 00:20:33 localhost poed[25224]: ERROR: Invalid config file format
Jul 09 00:20:33 localhost systemd[1]: Failed to start DentOS POE Agent.
```

Check the persistent config file under `/etc/poe_agent/poe_perm_cfg.json` or runtime config `/run/poe_runtime_cfg.json` format. Must be legal JSON format, match the config rule, and non-zero size.

If the configs are still not accepted by the PoE agent, delete the configs or move to another path for backup. Restart the agent and use `poecli cfg -s` to re-create as needed.

**Step-by-Step to bring the PoE agent back online:**

```
rm /etc/poe_agent/poe_perm_cfg.json /run/poe_runtime_cfg.json /run/poed.pid /run/poe_ipc_event
```

```
systemctl restart poed
```

Check agent log:

```
journalctl -u poed -n 15

Jul 09 00:25:33 localhost poed.py[1493]: INFO: Initialize PoE agent
Jul 09 00:25:33 localhost systemd[1]: Started DentOS POE Agent.
Jul 09 00:25:33 localhost poed.py[1493]: INFO: Load default config
Jul 09 00:25:33 localhost poed.py[1493]: INFO: PoE agent is running
```

The PoE agent simplifies the management of PoE chips, ensuring seamless operation and automatic restoration of configurations across various boot scenarios and system crashes. By following the commands and guidelines provided in this document, one can efficiently manage their PoE settings and troubleshoot common issues, ensuring a stable and reliable network environment.

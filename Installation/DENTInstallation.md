---
title: DENT Installation
parent: Installation
nav_order: 1
layout: default
---

# DENT Installation

## Overview

This section will describe the process of installing DENT on a device.

The DENT NOS build system uses an ONIE-compatible installer.
This section will describe installing the dentOS using a USB drive and connecting
to the device using a serial console.

Other alternative methods for installing with an ONIE
system can also be found here.

Steps:

1. Download or manually compile the installer image and place it on the USB drive
   under the name `onie-installer`
   (we assume that the working directory is the top directory in the local copy of the repository):

```
~ $ sudo mount /dev/sda /mnt/usb
~ $ sudo cp RELEASE/stretch/arm64/DENTOS-main_ONL-OS9_2021-10-26.1332-240e9a7_ARM64_INSTALLED_INSTALLER /mnt/usb/onie-installer
~ $ sudo umount /mnt/usb
```

Enter the ONIE environment Power on the device and upon seeing the prompt,
press `123<ENTER>` to stop autoboot and enter
the U-Boot environment. The output should be as follows:

```
U-Boot 2018.03-devel-1.2.0 (Jul 17 2020 - 13:58:51 +0800) TN48M/TN48M-P/TN4810M/TN48M2 V05

Model: Marvell Armada 7040 TN48M/TN48M-P/TN4810M/TN48M2
SoC: Armada7040-B0; AP806-B0; CP115-A0
Clock:  CPU     1400 [MHz]
	DDR     800  [MHz]
	FABRIC  800  [MHz]
	MSS     200  [MHz]
LLC Enabled (Exclusive Mode)
DRAM:  8 GiB
Comphy chip #0:
Comphy-0: PEX0          5 Gbps
Comphy-1: SATA0         6 Gbps
Comphy-2: SGMII0        3.125 Gbps
Comphy-3: SGMII1        1.25 Gbps
Comphy-4: UNCONNECTED
Comphy-5: SGMII2        1.25 Gbps
UTMI PHY 0 initialized to USB Host0
PCIE-0: Link up (Gen2-x1, Bus0)
NAND:  0 MiB
MMC:
Loading Environment from SPI Flash... Bus spi@700600 CS0 address is not set correctly.
SF: Detected w25q128bv with page size 256 Bytes, erase size 4 KiB, total 16 MiB
OK
EEPROM: TlvInfo v1 len=127
Model: Marvell Armada 7040 TN48M/TN48M-P/TN4810M/TN48M2
Net:   eth0: mvpp2-0, eth1: mvpp2-1, eth2: mvpp2-2 [PRIME]
SF: Detected w25q128bv with page size 256 Bytes, erase size 4 KiB, total 16 MiB
Type 123<ENTER> to STOP autoboot
```

If the device is currently running a non-ONIE OS operating system, first trigger
the ONIE environment from U-boot. U-Boot has a set of pre-defined environment variables,
out of which the most important one is `bootcmd`.

To trigger ONIE from U-Boot use the `onie_boot_reason` environment variable to
`install` and run bootcmd afterward.

```
Marvell>> setenv onie_boot_reason install
Marvell>> run bootcmd
Loading Open Network Install Environment ...
Platform: arm64-delta_tn48m-r0
Version: 2019.08
SF: Detected w25q128bv with page size 256 Bytes, erase size 4 KiB, total 16 MiB
device 0 offset 0x400000, size 0xc00000
```

The system will now boot into ONIE with Installer Mode Enabled. Expected console output is as follows:

```
** Installer Mode Enabled **
ONIE:/ #
ONIE:/ # down.
```

3. Plug the USB into the device USB port and mount it

`ONIE:/ # mount /dev/sdb1 /mnt/usb/`

The system should automatically locate the `onie-installer` file on the drive and execute it. After the installer
is executed, the device should boot into the dentOS environment. The default password for the `root` user is `onl`.

In case there is no pre-installed operating system on the device, powering up the switch will automatically boot
into the ONIE install environment. ONIE will scan the USB drive for the installer file and install the DentOS from it.
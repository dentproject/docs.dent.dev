---
title: GNS3 on a local machine
parent: Installation
nav_order: 2
layout: default
---

## Using your local machine

## Installation Steps

### Prerequisites

- [GNS3](https://docs.gns3.com/docs/) installed on your system. <br>

### 1. Download DENT NOS Files

&nbsp;&nbsp;&nbsp; Visit the DENT NOS repository on OneDrive to download the required files: [DENT NOS Files](https://onedrive.live.com/?authkey=%21AJV2rWTocq%5FG6KI&id=B4D5FD54A1A7D444%2144829&cid=B4D5FD54A1A7D444).

### 2. Uncompress Disk Image

&nbsp;&nbsp;&nbsp; Uncompress the downloaded disk image file.

### 3. Import Appliance to GNS3

![ImageTwoOfLocalUsage](../../Images/ImagesForGNS3/ImageTwoOfLocalUsage.png)

&nbsp;&nbsp;&nbsp; a. Open GNS3 and go to `File -> Import Appliance`. <br>
&nbsp;&nbsp;&nbsp; b. Select the GNS3 appliance file _(gns3a file)_ you downloaded from the OneDrive link. <br>
&nbsp;&nbsp;&nbsp; c. Choose the server on which to run the appliance.

![ImageThreeOfLocalUsage](../../Images/ImagesForGNS3/ImageThreeOfLocalUsage.png)

### 4. Choose QEMU Binary

&nbsp;&nbsp;&nbsp; a. Choose the QEMU binary that will be used to run the DENT NOS appliance. <br>
&nbsp;&nbsp;&nbsp; b. The recommended option is `/bin/qemu-system-x86_64 (v8.0.4)`.

![ImageFourOfLocalUsage](../../Images/ImagesForGNS3/ImageFourOfLocalUsage.png)

### 5. Import DENT NOS Image

&nbsp;&nbsp;&nbsp; a. Click on the DENT NOS image file and import it. <br>
&nbsp;&nbsp;&nbsp; b. Wait for the upload to finish; this may take some time.

![ImageFiveOfLocaLUsage](../../Images/ImagesForGNS3/ImageFiveOfLocalUsage.png)

### 6. Confirm Installation

&nbsp;&nbsp;&nbsp; a. You will be prompted with an installation confirmation. <br>
![ImageSixOfLocaLUsage](../../Images/ImagesForGNS3/ImageSixOfLocalUsage.png) <br>
&nbsp;&nbsp;&nbsp; b. Click “Yes” to confirm the installation. <br>

![ImageTenOfLocaLUsage](../../Images/ImagesForGNS3/ImageTenOfLocalUsage.png)

&nbsp;&nbsp;&nbsp; **Congratulations!** <br>
&nbsp;&nbsp;&nbsp; You have successfully installed DENT NOS on GNS3.

### Start Using DENT NOS in GNS3

&nbsp;&nbsp;&nbsp; a. Drag the DENT NOS appliance into the main window of your GNS3 project. <br>
&nbsp;&nbsp;&nbsp; b. Create your network topology, adding DENT NOS appliances as needed. <br>
&nbsp;&nbsp;&nbsp; c. Right-click on each appliance and select “Start” to initiate the simulation.

![ImageSevenOfLocaLUsage](../../Images/ImagesForGNS3/ImageSevenOfLocalUsage.png)

### Default Credentials

&nbsp;&nbsp;&nbsp; DENT login: root <br>
&nbsp;&nbsp;&nbsp; Password: onl <br>

![ImageEightOfLocaLUsage](../../Images/ImagesForGNS3/ImageEightOfLocalUsage.png)

<div style="border-top: 1px solid black;"></div>

For more information, you can visit [dent.dev](https://dent.dev).

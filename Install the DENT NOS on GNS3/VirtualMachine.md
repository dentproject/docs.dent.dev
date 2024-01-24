---
title: Using a virtual machine to run the DENT NOS in GNS3
parent: Install the DENT NOS on GNS3
nav_order: 2
---

## Using a virtual machine to run the DENT NOS in GNS3


First, install the DENT GNS3 appliance file as well as the disk
image for the dent Virtual Machine. You can find the required files
here:** [DENT Image and gns3a file](https://1drv.ms/f/s!AkTUp6FU_dW0gt4dlXatZOhyr8boog?e=Ltqpa5.)

![ImageOneOfVmUsage.png](../Images/ImagesForGNS3/ImageOneOfVmUsage.png)

Install the appropirate GNS3 VM for your machine

![ImageTwoOfVmUsage.png](../Images/ImagesForGNS3/ImageTwoOfVmUsage.png)

If you are using VMware Workstation Pro, install the VMWare
Workstation and Fusion GNS3 VM and extract the .zip folder you
downloaded.

![ImageThreeOfVmUsage.png](../Images/ImagesForGNS3/ImageThreeOfVmUsage.png)

Open VMWare Workstation then click Open a Virtual Machine and
select the extracted GNS3 VMWare Workstation folder.

![ImageFourOfVmUsage.png](../Images/ImagesForGNS3/ImageFourOfVmUsage.png)

When ready, run the Virtual Machine. You should see a screen
similar to this below once the Virtual machine is running.

![ImageFiveOfVmUsage.png](../Images/ImagesForGNS3/ImageFiveOfVmUsage.png)

Now Open GNS3, go to Edit -> Preferences -> GNS3 VM and check
the “Enable the (GNS3) VM” box. Select the appropriate Virtual
Machine you are running GNS3 on as your Virtualization Engine
and click ok

![ImageSixOfVmUsage.png](../Images/ImagesForGNS3/ImageSixOfVmUsage.png)

You Should now See an Additional Server Listed under Server Summary

![ImageSevenOfVmUsage.png](../Images/ImagesForGNS3/ImageSevenOfVmUsage.png)

Go to File -> Import Appliance and select the appliance file.
In this scenario we will select one of the previously downloaded
files “DENT - 3.2”.

![ImageEightOfVmUsage.png](../Images/ImagesForGNS3/ImageEightOfVmUsage.png)

The QEMU binary that will be used to run this appliance is
recommended as /bin/qemu-system-x86_64(v4.2.1).

![ImageNineOfVmUsage.png](../Images/ImagesForGNS3/ImageNineOfVmUsage.png)

Next, we need to import the DENT image file by selecting again
one of the previously downloaded files “dent-vm.qcow2” and clicking
import.

**Wait for the upload to finish, it may take some time.**

![ImageTenOfVmUsage.png](../Images/ImagesForGNS3/ImageTenOfVmUsage.png)

Once the upload is finished, you may click next and yes to
install DENT

![ImageElevenOfVmUsage.png](../Images/ImagesForGNS3/ImageElevenOfVmUsage.png)

Once Installed you may now use the DENT appliance in GNS3.
The example below demonstrate 3 dent appliances connecting to
each other

![ImageTwelveOfVmUsage.png](../Images/ImagesForGNS3/ImageTwelveOfVmUsage.png)

After Starting the simulation you may right-click on any DENT
appliance and select _console_ to log-in.

**The default credentials are:**
- **Localhost login: root**
- **Password: onl**

![ImageThirteenOfVmUsage.png](../Images/ImagesForGNS3/ImageThirteenOfVmUsage.png)

### You have now successfully set up DENT in GNS3 with a virtual machine

For more information on how to set up DENT in GNS3 with a virtual
machine feel free to contact the author Korel Ucpinar at
Korelucpinar@gmail.com

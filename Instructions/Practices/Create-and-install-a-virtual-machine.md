# Practice: Create and install a virtual machine

## Required VMs

* VN1-SRV1
* PM-SRV1
* CL1

## Task

On PM-SRV1, create a new virtual machine with the name PM-SRV20. The virtual machine should have 1 GB of memory, 4 virtual processor, time synchronization disabled, and should install Windows Server Core from 2025_x64_EN_Eval.iso.

> Why would you want to deactivate time synchronization?

## Instructions

Perform these steps on CL1.

1. Sign in as **ad\Administrator**.
1. Open **Hyper-V Manager**.
1. In Hyper-V Manager, click **PM-SRV1**.
1. In the context-menu of **PM-SRV1**, click **New**, **Virtual Machine...**
1. In New Virtual Machine Wizard, on page Before You Begin, click **Next >**.
1. On page Specify name and Location, in **Name**, type **PM-SRV20** and click **Next >**.
1. On page Specify Generation, click **Generation 2** and click **Next >**.
1. On page Assign Memory, in **Startup Memory**, type **1024** and click **Next >**.
1. On page Configure Networking, in **Connection**, click **External** and click **Next >**.
1. On page Connect virtual hard disk, ensure **Create a virtual hard disk** is selected. In **Name**, ensure **PM-SRV20.vhdx**, and, in location **C:\\hyper-v\\Virtual Hard Disks\\** is filled in. Click **Next >**.
1. On page Installation Options, click **Install an operating system from a bootable image file** and click **Browse...**
1. In Open, expand **pm-srv1.ad.adatum.com**, **Local Disk (C:)** and click **LabResources**.
1. Click **2025_x64_EN_Eval.iso** and click **Open**.
1. In **New Virtual Machine Wizard**, on page **Installation Options**, click **Next >**.
1. On page Summary, click **Finish**.
1. In **Hyper-V Manager**, under **Virtual machines**, in the context-menu of **PM-SRV20**, click **Settings...**.
1. In **Settings for PM-SRV20 on PM-SRV1**, click **Security**.
1. Under **Encryption Support**, activate **Enable Trusted Platform Module** and activate **Encrypt state and virtual machine migration traffic**.
1. Click **Processor**.
1. Under Processor, in **Number of virtual processors**, type **4**.
1. Click **Integration Services**.
1. Under Integration Services, deactivate **Time synchronization**.

    > On domain-joined computers, it is recommended to disable time synchronization, because they synchronize the time with a domain controller.

1. Click **OK**.
1. In **Hyper-V Manager**, under **Virtual machines**, in the context-menu of **PM-SRV20**, click **Connect...**.
1. In PM-SRV20 on PM-SRV1 - Virtual Machine Connection, in the menu, click **Action**, **Start**.
1. On the message Press any key to boot from CD or DVD, press any key within a few seconds. If fail to do so, and the VM tries to start PXE over IPv4, on the menu, click **Action**, **Reset...**.
1. In Windows Server Setup, on page Select language settings, configure **Time and currency format**  as you wish and click **Next**.
1. On page Select keyboard settings, configure the **Keyboard or input method** as you wich and click **Next**.
1. On page Select setup option, ensure **Install Windows Server** is selected, click **I agree everything will be deleted including files, apps, and settings** and click **Next**.
1. On page Select image, click **Windows Server 2025 Datacenter Evaluation** and click **Next**.
1. On page Applicable notices and license terms, click **Accept**.
1. On page Select location to install Windows Server, explore the options and click **Next**.
1. On page Ready to install, click **Install**.

Do not wait for the installation to finish.

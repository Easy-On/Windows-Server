# Practice: Create a custom Microsoft Management Console

## Required VMs

* VN1-SRV1
* CL1

## Setup

If you skipped the practice [Install Remote Server Administration Tools](Install-Remote-Server-Administration-Tools.md), on CL1, run ````C:\LabResources\Solutions\Install-RemoteServerAdministrationTools.ps1````.

## Task

On CL1, create a custom Microsoft Management Console with the snap-ins Active Directory Users and Computers and Computer Management and save it to the desktop of all users.

## Instructions

Perform these steps on CL1.

1. Logon as **ad\Administrator**.
1. Open **File Explorer**.
1. In File Explorer, click **View**, **Show**, **Hidden items**.
1. Close File Explorer.
1. Run **mmc**.
1. On the menu, click **File**, **Add/Remove Snap-in...**
1. In Add or Remove Snap-Ins, under **Available snap-ins**, click **Active Directory Users and Computers** and click **Add >**.
1. Click **Computer Management** and click **Add >**.
1. In Computer Management, make sure, **Local computer** is selected and click **Finish**.
1. Click **OK**.
1. In the context menu of **Console Root**, click **Rename**.
1. Type **Basic Administration** and press ENTER.
1. On the menu, click **File**, **Save As...**.
1. Save the console as **C:\Users\Public\Desktop\Basic Administration.msc**.

   *Note*: In the dialog, the folder Desktop is displayed as **Public Desktop**.

1. Close the console.

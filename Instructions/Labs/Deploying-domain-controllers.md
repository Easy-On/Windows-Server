# Lab: Deploying domain controllers

## Required VMs

* VN1-SRV1
* VN1-SRV4
* VN1-SRV5
* VN2-SRV1
* VN2-SRV2
* CL1
* CL3

## Setup

1. On **CL1**, sign in as **ad\\Administrator**.
1. On **CL3**, sign in as **.\\Administrator**.
1. On **VN1-SRV1** sign in as **ad\\Administrator**.
1. On **VN2-SRV2** sign in as **.\\Administrator**.

You must have completed the practice [Explore Server Manager](../Practices/Explore-Server-Manager.md). If you skipped the practice, on **CL1**, in Terminal, run ````C:\LabResources\Solutions\Add-ServerManagerServers.ps1````.

You must have completed the practice [Install Windows Admin Center using a script](../Practices/Install-Windows-Admin-Center-using-a-script.md). If you skipped the practice, on **VN1-SRV4**, sign in as **ad\Administrator**, and run ````C:\LabResources\Solutions\Install-AdminCenter.ps1````.

## Introduction

The domain controller still running Windows Server 2019 must be replaced by a Windows Server 2022 domain controller. Moreover, Adatum is expanding to a new location. An additional domain controller must be installed at the new location. Furthermore, Adatum launches a new subsidiary with the name Contoso. Because it is expected, that the subsidiary will be sold soon, a new forest needs to be created for the subsidiary.

## Exercises

1. [Deploy additional domain controllers](#exercise-1-deploy-additional-domain-controllers)
1. [Check domain controller health](#exercise-2-check-domain-controller-health)
1. [Transfer flexible single master operation roles](#exercise-3-transfer-flexible-single-master-operation-roles)
1. [Decommission a domain controller](#exercise-4-decommission-a-domain-controller)
1. [Raise domain and forest functional level](#exercise-5-raise-the-domain-and-forest-functional-level)
1. [Enable database 32K pages](#exercise-6-enable-database-32k-pages)
1. [Deploy a new forest](#exercise-7-deploy-a-new-forest)

Note: Exercise 6 is not dependent on the other exercises. To safe time, you may run the tasks of exercise 6 while you are waiting for execution of tasks in the other exercises.

## Exercise 1: Deploy additional domain controllers

1. [Install the Remote Server Administration DNS Server Tools](#task-1-install-the-remote-server-administration-dns-server-tools) on CL1
1. [Install Active Directory Domain Services](#task-2-install-active-directory-domain-services) on VN1-SRV5 and VN2-SRV1
1. [Configure Active Directory Domain Services as a additional domain controller in an existing domain](#task-3-configure-active-directory-domain-services-as-an-additional-domain-controller-in-an-existing-domain) on VN1-SRV5 and VN2-SRV1
1. [Configure forwarders](#task-4-configure-forwarders) on VN1-SRV5 and VN2-SRV1 to 8.8.8.8 and 8.8.4.4
1. [Configure DNS client settings](#task-5-configure-dns-client-settings) on VN1-SRV5 to use VN2-SRV1 as preferred DNS server

Note: It is recommended to use another domain controller as DNS server. However, in real world, you should choose a DNS server on the same network.

### Task 1: Install the Remote Server Administration DNS Server Tools

#### Desktop experience

Perform these steps on CL1.

1. Open **Settings**.
1. In Settings, click **System**.
1. In System, click **Optional features**.
1. In Add an optional feature, in the text field **Find an available optional feature**, type **RSAT**.
1. Activate the check box beside **RSAT: DNS Server Tools**.
1. Click **Next**.
1. Click **Install**.
1. If required, restart the computer.

You do not need to wait for the completion of the installation

#### PowerShell

Perform these steps on CL1.

1. In the context menu of **Start**, click **Terminal (Admin)**.
1. Add the windows capabilities **RSAT: Server DNS Server tools**.

    ````powershell
    Get-WindowsCapability -Online -Name 'Rsat.Dns.Tools*' |
    Add-WindowsCapability -Online
    ````

You do not need to wait for the completion of the installation.

### Task 2: Install Active Directory Domain Services

#### Desktop experience

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, in the menu, click **Manage**, **Add Roles and Features**.
1. In Add Roles and Features Wizard, on page Before You Begin, click **Next >**.
1. On page Installation Type, ensure **Role-based or feature-based installation** is selected and click **Next >**.
1. On page Server Selection, click **VN1-SRV5** and click **Next >**.
1. On page Server Roles, activate **Active Directory Domain Services**.
1. In the dialog **Add features that are required for Active Directory Domain Services?**, click **Add Features**
1. On page **Server Roles**, click **Next >**.
1. On page Features, click **Next >**.
1. On page **AD DS**, click **Next >**.
1. On page **Confirmation**, activate the checkbox **Restart the destination server automatically if required** and click **Install**.
1. On page **Results**, click **Close**.

Repeat from step 2, but in step 5, on page **Server Selection**, click **VN2-SRV1**.

#### Windows Admin Center

Perform this task on CL1.

1. Open **Microsoft Edge**.
1. In Microsoft Edge, navigate to <https://admincenter.ad.adatum.com/>
1. On the Windows Admin Center page, click **vn1-srv5.ad.adatum.com**.
1. On the page vn1-srv5.ad.adatum.com, unter Tools, click **Roles & features**.
1. Under Roles and features, click **Active Directory Domain Services** and click **Install**.
1. In the pane Install Role and Features, activate the checkbox **Reboot the server automatically if required**, and click **Yes**.
1. At the top-left, click **Windows Admin Center**.

Repeat from step 3 for **vn2-srv1.ad.adatum.com**.

#### PowerShell

Peform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Install the windows feature **Active Directory Domain Services** on **VN1-SRV5** and **VN2-SRV1**.

    ````powershell
    Invoke-Command -ComputerName VN1-SRV5, VN2-SRV1 -ScriptBlock {
        Install-WindowsFeature `
            -Name AD-Domain-Services -IncludeManagementTools -Restart
    }
    ````

### Task 3: Configure Active Directory Domain Services as an additional domain controller in an existing domain

#### Desktop experience

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, click *Notifications* (the flag with the yellow warning triangle), and under the message **Configuration required for Active Directory Domain Services at VN1-SRV5**, click **Promote this server to a domain controller**.

    If you do not see a notification, click *Refresh*.

1. In Active Directory Domain Services Configuration Wizard, on page Deployment Configuration, ensure **Add a domain controller to an existing domain** is selected. In **Domain**, ensure **ad.adatum.com** is filled in. Beside **\<No credentials provided\>**, click **Change...**.
1. In the dialog Credentials for deployment operation, enter the credentials for **Administrator@ad.adatum.com** and click **OK**.
1. On page **Deployment Configuration**, click **Next >**.
1. On page **Domain Controller Options**, ensure **Domain Name System (DNS) server** and **Global Catalog (GC)** are activated. Under **Type the Directory Services Restore Mode (DSRM) password**, in **Password** and **Confirm password**, type a secure password and take a note. You will need the password for a later lab. Click **Next >**.
1. On page DNS Options, click **Next >**.
1. On page **Additional Options**, click **Next >**.
1. On page **Paths**, click **Next >**.

    Note: In real world, it is recommended to have the paths on a separate drive.

1. On page Review Options, click **Next >**.
1. On page Prerequisites Check, click **Install**.
1. On page Results, click **Close**.

Repeat from step 2 to promote VN2-SRV1 to a domain controller.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Store the password in a variable.

    ````powershell
    $username = "Administrator@ad.adatum.com"
    $securePassword = Read-Host `
        -Prompt `
            "Password for $(
                $username
            ) (will also be the Directory Services Restore Mode (DSRM) password)" `
        -AsSecureString
    ````

1. When prompted, enter the credentials for **Administrator@ad.adatum.com**.
1. Promote **VN1-SRV5** and **VN2-SRV1** to domain controllers in the domain **ad.adatum.com**. Install DNS at the same time.

    ````powershell
    # Convert the secure string back to a plain text string

    $password = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto(
        [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR(
            $securePassword
        )
    ) 

    $job = Invoke-Command -ComputerName VN1-SRV5, VN2-SRV1 -AsJob -ScriptBlock {
        # Convert the password into a secure string

        $securePassword = `
            ConvertTo-SecureString -String $using:password -AsPlainText -Force
        

        $safeModeAdministratorPassword = $securePassword

        # Create credentials
        $credential = New-Object `
            -TypeName pscredential `
            -ArgumentList $using:username, $securePassword
        
        Install-ADDSDomainController `
            -DomainName ad.adatum.com `
            -Credential $credential `
            -SafeModeAdministratorPassword $safeModeAdministratorPassword `
            -InstallDns `
            -Force
    }
    ````

1. Wait for the job to complete.

    ````powershell
    $job | Wait-Job
    ````

    This will take a few minutes.

1. Read the output of the job.

    ````powershell
    $job | Receive-Job
    ````

    The value of the property **Status** should be **Success** for both servers.

Wait for the restart of VN1-SRV5 and VN2-SRV1 to complete.

### Task 4: Configure forwarders

#### Desktop experience

Perform this task on CL1.

1. Open **DNS**.
1. In **Connect to DNS Server**, click **The following computer**, type **VN1-SRV5.ad.adatum.com**, and click **OK**.
1. In DNS Manager, click **VN1-SRV5.ad.adatum.com**.
1. In vn1-srv5.ad.adatum.com, double-click **Forwarders**.
1. In vn1-srv5.ad.adatum.com Properties, on tab Forwarders, click **Edit...**
1. In Edit Forwarders, click **10.1.1.8** and click **Delete**.
1. In **\<Click here to add an IP Address or DNS Name\>**, enter **8.8.8.8**. Repeat this step with **8.8.4.4** and click **OK**.
1. In **VN1-SRV5.ad.adatum.com Properties**, click **OK**.
1. In **DNS Manager**, in the context-menu of **DNS**, click **Connect to DNS Server...**
1. In **Connect to DNS Server**, click **The following computer**, type **VN2-SRV1.ad.adatum.com**, and click **OK**.

Repeat from step 3 for **vn2-srv1.ad.adatum.com**.

#### PowerShell

Perform this task on CL1.

1. Run **Terminal**.
1. In Terminal, configure the forwarder for DNS server **VN1-SRV5** and **VN2-SRV1** to **8.8.8.8** and **8.8.4.4**.

    ````powershell
    Set-DnsServerForwarder -IPAddress 8.8.8.8, 8.8.4.4  -ComputerName VN1-SRV5
    Set-DnsServerForwarder -IPAddress 8.8.8.8, 8.8.4.4  -ComputerName VN2-SRV1
    ````

### Task 5: Configure DNS client settings

#### SConfig

Perform this task on VN1-SRV5.

1. Sign in as **ad\administrator**.
1. In SConfig, enter **8**.
1. In Network settings, enter then Index # of the network adapter with the IP address starting with 10.1.1, e.g. 1.
1. In Network adapter settings, enter **2**.
1. Beside Enter new preferred DNS server, enter **10.1.2.8**.
1. Beside Enter alternate DNS server, enter **127.0.0.1**.
1. Press ENTER to continue.
1. In SConfig, enter **12**.
1. Beside Are you sure you want to log off, enter **y**.

#### Windows Admin Center

Perform this task on CL1.

1. Open **Microsoft Edge**.
1. In Microsoft Edge, navigate to <https://admincenter.ad.adatum.com/>
1. On the Windows Admin Center page, click **vn1-srv5.ad.adatum.com**.
1. On the page vn1-srv5.ad.adatum.com, under Tools, click **Networks**.
1. Under Networks, click **VNet1** and click **Settings**.
1. On the tab IPv4, under Use the following DNS server addresses, under **Preferred DNS Server**, type **10.1.2.8**. Under Alternate DNS server, type **127.0.0.1**.
1. Click **Save**.
1. Click **Close**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Create a CIM session to **VN1-SRV5**.

    ````powershell
    $cimSession = New-CimSession -ComputerName VN1-SRV5
    ````

1. Set the DNS client server address for **VN1-SRV5** to **10.1.2.8** and **127.0.0.1**.

    ````powershell
    Set-DnsClientServerAddress `
        -InterfaceAlias VNet1 `
        -ServerAddresses 10.1.2.8, 127.0.0.1 `
        -CimSession $cimSession
    ````

1. Close and remove the CIM session.

    ````powershell
    Remove-CimSession $cimSession
    ````

## Exercise 2: Check domain controller health

1. [Verify DNS entries for Active Directory](#task-1-verify-dns-entries-for-active-directory)

    > Which records were created in the DNS zone _msdcs.ad.adatum.com?

    > Which SRV records were created in the DNS zone ad.adatum.com?

1. [Verify shares for Active Directory](#task-2-verify-shares-for-active-directory)

    > Which shares where created by the configuration of Active Directory Domain Services on VN1-SRV5 and VN2-SRV1?

1. [Verify the health of AD DS](#task-3-verify-the-health-of-ad-ds)

    > Are there any warnings from the Best Practices Analyzer?

### Task 1: Verify DNS entries for Active Directory

Perform this task on CL1.

1. Open **DNS**.
1. If the dialog **Connect to DNS Server** appears, click **The following computer**, type **VN1-SRV5**, and click **OK**.
1. In DNS, click **VN1-SRV5**.
1. Expand **VN1-SRV5**, **Forward Lookup Zones** and click **_msdcs.ad.adatum.com**

    > There should be 3 CNAME records, pointing to VN1-SRV1.ad.adatum.com, VN1-SRV5.ad.adatum.com, and vn2-srv1.ad.adatum.com.

1. Expand **ad.adatum.com**, and click **_tcp**.

    > There should 12 SRV records for the services \_gc, \_kerberos, \_kpasswd, and \_ldap, pointing to VN1-SRV1.ad.adatum.com, VN1-SRV5.ad.adatum.com, and vn2-srv1.ad.adatum.com.

If any records, are missing, open **Terminal** and execute the following command:

````powershell
Invoke-Command -ComputerName VN1-SRV5, VN2-SRV1 { Restart-Service Netlogon }
````

Wait a minute and then check again. If the problem persists, ask the instructor.

### Task 2: Verify shares for Active Directory

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server manager, click **File and Storage Services**.
1. In File and Storage Services, click **Shares**

    > The shares NETLOGON and SYSVOL should be present on VN1-SRV5 and VN2-SRV1.

### Task 3: Verify the health of AD DS

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, click **Dashboard**.
1. On Dashboard, ensure that under **ROLES AND SERVER GROUPS**, beside **AD DS** and **DNS**, the number **3** is written and there is a green arrow pointing up under these tiles.
1. In the left pane, click **AD DS**. Under **EVENTS**, there should be no error events from the current day (there might me older error events).
1. Under **BEST PRACTICES ANALYZER**, click **TASKS**, **Start BPA Scan**.
1. In the dialog Select Servers, activate all three domain controllers and click **Start Scan**.

    > Review any warnings or errors, if present.

If time permits, you can try to fix the warning and errors and run the the BPA scan again.

## Exercise 3: Transfer flexible single master operation roles

1. [Transfer the domain-wide flexible single master operation roles](#task-1-transfer-the-domain-wide-flexible-single-master-operation-roles)
1. [Transfer the forest-wide flexible single master operation roles](#task-2-transfer-the-forest-wide-flexible-single-master-operation-roles)

### Task 1: Transfer the domain-wide flexible single master operation roles

#### Desktop experience

Perform this task on CL1.

1. Open **Active Directory Users and Computers**.
1. In Active Directory Users and Computers, in the context-menu of **ad.adatum.com**, click **Change Domain Controller...**.
1. In Change Directory Server, click **VN1-SRV5.ad.adatum.com** and click **OK**.
1. In the context-menu of **ad.adatum.com**, click **Operations Masters...**
1. In Operations Masters, on the tab **RID**, click **Change...**
1. In the message box **Are you sure you want to transfer the operations master role?**, click **Yes**.
1. In the message box **The operations master role was successfully transferred.**, click **OK**.
1. In **Operations Masters**, click the tab **PDC**.
1. On the tab **PDC**, click **Change...**
1. In the message box **Are you sure you want to transfer the operations master role?**, click **Yes**.
1. In the message box **The operations master role was successfully transferred.**, click **OK**.
1. In **Operations Masters**, click the tab **Infrastructure**.
1. On the tab **Infrastructure**, click **Change...**
1. In the message box **Are you sure you want to transfer the operations master role?**, click **Yes**.
1. In the message box **The operations master role was successfully transferred.**, click **OK**.
1. In **Operations Masters**, click **Close**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Move the roles **RID master**, **infrastructure master**, and **PDC emulator** to **VN1-SRV5**.

    ````powershell
    Move-ADDirectoryServerOperationMasterRole `
        -Identity VN1-SRV5 `
        -OperationMasterRole RIDMaster, InfrastructureMaster, PDCEmulator
    ````

1. At the prompt **Do you want to move role 'RIDMaster' to server 'VN1-SRV5.ad.adatum.com' ?**, enter **y**.
1. At the prompt **Do you want to move role 'InfrastructureMaster' to server 'VN1-SRV5.ad.adatum.com' ?**, enter **y**.
1. At the prompt **Do you want to move role 'PDCEmulator' to server 'VN1-SRV5.ad.adatum.com' ?**, enter **y**.

### Task 2: Transfer the forest-wide flexible single master operation roles

#### Desktop experience

Perform this task on CL1.

1. Open **Active Directory Domains and Trusts**.
1. In Active Directory Domains and Trusts, in the context-menu of **Active Directory Domains and Trusts**, click **Change Active Directory Domain Controller...**
1. In Change Directory Server, click **VN1-SRV5.ad.adatum.com** and click **OK**.
1. In the context-menu of **Active Directory Domains and Trusts**, click **Operations Master...**
1. In Operations Master, click **Change...**
1. In the message box **Are you sure you want to transfer the operations master role to a different computer?**, click **Yes**.
1. In the message box **The operations master role was successfully transferred.**, click **OK**.
1. In **Operations Master**, click **Close**.
1. In the context menu of **Start**, click **Terminal (Admin)**.
1. Register the Schema Management snap-in.

    ````powershell
    regsvr32.exe C:\Windows\System32\schmmgmt.dll
    ````

1. In the message box **DllRegisterServer in C:\Windows\System32\schmmgmt.dll succeeded.**, click **OK**.
1. Open an empty MMC.

    ````powershell
    mmc.exe
    ````

1. In Console1 - [Console Root], in the menu, click **File**, **Add /Remove Snap-In...**
1. In Add or Remove Snap-Ins, click **Active Directory Schema**, click **Add >**, and click **OK**.
1. In the context-menu of **Active Directory Schema**, click **Change Active Directory Domain Controller...**
1. In Change Directory Server, click **VN1-SRV5.ad.adatum.com** and click **OK**.
1. In the message box **Active Directory Schema snap-in is not connected to the schema operations master. You will not be able to perform any changes. Schema modifications can only be made on the schema FSMO holder.**, click **OK**.

1. In the context-menu of **Active Directory Schema**, click **Operations Master...**
1. In Operations Master, click **Change**.
1. In the message box **Are you sure you want to change the Operations Master?**, click **Yes**.
1. In the message box **Operations Master successfully transferred.**, click **OK**.
1. In **Operations Master**, click **Close**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Move the roles **domain naming master** and **schema master** to **VN1-SRV5**.

    ````powershell
    Move-ADDirectoryServerOperationMasterRole `
        -Identity VN1-SRV5 `
        -OperationMasterRole DomainNamingMaster, SchemaMaster `
        -Confirm:$false
    ````

## Exercise 4: Decommission a domain controller

1. [Change the DNS client server addresses](#task-1-change-the-dns-client-server-addresses) on CL1 to 10.1.1.40.
1. [Change the IP address of the domain controller to decommission](#task-2-change-the-ip-address-of-the-domain-controller-to-decommission) VN1-SRV1 to 10.1.1.9 and the DNS client server addresses to 10.1.1.40 and 10.1.2.8
1. [Update the Host (A) record of the domain controller to decommission] VN1-SRV1 to 10.1.1.9
1. [Add the IP address of the decommissioned domain controller to the new domain controller](#task-4-add-the-ip-address-of-the-decommissioned-domain-controller-to-the-new-domain-controller): Add 10.1.1.8 to VN1-SRV5
1. [Demote the old domain controller](#task-5-demote-the-old-domain-controller) VN1-SRV1
1. [Remove roles from the decommissioned domain controller](#task-6-remove-roles-from-the-decommissioned-domain-controller) VN1-SRV1

*Note:* In this exercise, we add the IP address of the decommissioned domain controller to the new domain controller, so we do not have to reconfigure the DNS client settings on the other computers on the network. If all computers use DHCP, you could reconfigure the DHCP option DNS server instead. You would do this before task 1 and then wait for the DHCP lease period to expire before proceeding. Moreover, you would skip task 2.

### Task 1: Change the DNS client server addresses

#### Desktop experience

Perform this task on CL1.

1. Open **Settings**.
1. In Settings, in the left pane, click **Network & internet**.
1. In Network & internet, click **Ethernet**.
1. In Ethernet, beside **DNS server assignment**, click **Edit**.
1. In Edit IP Settings, under **Preferred DNS**, type **10.1.1.40**. Under **Alternate DNS**, type **10.1.2.8**. Click **Save**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal (Admin)**.
1. Set the DNS client server address on the interface **Ethernet** to **10.1.1.40** and **10.1.2.8**.

    ````powershell
    Set-DnsClientServerAddress `
        -InterfaceAlias Ethernet -ServerAddresses 10.1.1.40, 10.1.2.8
    ````

### Task 2: Change the IP address of the domain controller to decommission

#### SConfig

Perform this task on VN1-SRV1.

1. Run SConfig.

    ````shell
    SConfig
    ````

1. In sconfig, enter **8**.
1. In Network settings, enter **1**.
1. In Network Adapter settings, enter **1**.
1. Beside Select DHCP, Static IP, enter **s**.
1. Beside Enter static IP address, enter **10.1.1.9**.
1. Beside Enter subnet mask, enter **255.255.255.0**.
1. Beside Enter default gateway, enter **10.1.1.1**.

    If you receive an error message, skip to step 13 and change the IP address using one of the other methods provided.

1. In Network Adapter Settings, enter **2**.
1. Beside Enter new preferred DNS server, enter **10.1.1.40**.
1. In the message box Preferred DNS server set, click **OK**.
1. Beside Enter alternate DNS server, enter **10.1.2.8**.
1. In the message box Alternate DNS server set, click **OK**.
1. In Network Adapter Settings, enter **4**.
1. In Server Configuration, enter **12**.
1. In the message box **Are you sure to log off?**, click **Yes**.

#### Windows Admin Center

Perform this task on CL1.

1. Open **Microsoft Edge**.
1. In Microsoft Edge, navigate to <https://admincenter.ad.adatum.com/>
1. On the Windows Admin Center page, click **vn1-srv1.ad.adatum.com**.
1. On the page vn1-srv1.ad.adatum.com, under Tools, click **Networks**.
1. Under Networks, click **Ethernet** and click **Settings**.
1. On the tab IPv4, under Use the following IP address, under **IP Address**, type **10.1.1.9**. Under Use the following DNS server addresses, under **Preferred DNS Server**, type **10.1.1.40**. Under **Alternate DNS Server**, type **10.1.2.8**.
1. Click **Save**.
1. Click **Close**.

Note: At this point, Windows Admin Center might stop working for you, because the DNS client server IP address of CL1 now points to an IP address not responsive anymore.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Create a CIM session to **VN1-SRV1**.

    ````powershell
    $computerName = 'VN1-SRV1'
    $cimSession = New-CimSession -ComputerName $computerName
    ````

1. Find the network interface of the old IP address.

    ````powershell
    $oldIPAddress = '10.1.1.8'
    $interfaceAlias = (
        Get-NetIPAddress -CimSession $cimSession |
        Where-Object { $PSItem.IPAddress -eq $oldIPAddress }
    ).InterfaceAlias
    ````

1. Add the IP address **10.1.1.9**. with the prefix length of **24** to VN1-SRV1.

    ````powershell
    New-NetIPAddress `
        -InterfaceAlias $interfaceAlias `
        -IPAddress 10.1.1.9 `
        -PrefixLength 24 `
        -CimSession $cimSession
    ````

1. Set the DNS server addresses for VN1-SRV1 to **10.1.1.40** and **10.1.2.8**.

    ````powershell
    Set-DnsClientServerAddress `
        -InterfaceAlias $interfaceAlias `
        -ServerAddresses 10.1.1.40, 10.1.2.8 `
        -CimSession $cimSession
    ````

1. Remove the CIM session.

    ````powershell
    Remove-CimSession $cimSession
    ````

1. Remove the A record **VN1-SRV1** with the IP address **10.1.1.8** from DNS.

    ````powershell
    Remove-DnsServerResourceRecord `
        -ComputerName VN1-SRV5 `
        -ZoneName ad.adatum.com `
        -RRType A `
        -Name $computerName `
        -RecordData $oldIPAddress `
        -Force
    ````

1. Clear the DNS client cache.

    ````powershell
    Clear-DnsClientCache
    ````

1. Create a CIM session to **VN1-SRV1**.

    ````powershell
    $cimSession = New-CimSession -ComputerName $computerName
    ````

1. Remove the IP address **10.1.1.8** from VN1-SRV1.

    ````powershell
    Remove-NetIPAddress `
        -InterfaceAlias $interfaceAlias `
        -IPAddress $oldIPAddress `
        -CimSession $cimSession `
        -Confirm:$false
    ````

1. Remove the CIM session.

    ````powershell
    Remove-CimSession $cimSession
    ````

### Task 3: Update the Host (A) record of the domain controller to decommission

#### Desktop Experience

Perform this task on CL1.

1. Open **DNS**.
1. If the dialog **Connect to DNS Server** appears, click **The following computer**, type **VN1-SRV5**, and click **OK**.
1. In DNS, click **VN1-SRV5**.
1. Expand **VN1-SRV5**, **Forward Lookup Zones** and click **ad.adatum.com**
1. In the zone ad.adatum.com, double-click the record with the nae **vn1-srv1** and the type **Host (A)**.
1. In vn1-srv1 Properties, under **IP address**, type **10.1.1.9** and click **OK**.

#### Windows Admin Center

Perform this task on CL1.

1. Open **Microsoft Edge**.
1. In Microsoft Edge, navigate to **VN1-SRV9.ad.adatum.com**.
1. If necessary, install the extension **DNS**.

    1. Click *Settings* (the gear icon) in the top-right corner.
    1. In Settings, click **Extensions**.
    1. In Extensions click **DNS**.
    1. Click **Install**.

1. In Windows Admin Center, click **VN1-SRV5.ad.adatum.com**.
1. In the VN1-SRV5.ad.adatum.com, unter Tools, click **DNS**. If you do not see DNS there, go to step 3.
1. If you see a message The DNS PowerShell tools (RSAT) are not installed, click **Install**.

    Wait for the installation to complete. The page will automatically reload.

1. In DNS, click the zone **ad.adatum.com**.
1. In the Records pane, click the record with the name **vn1-srv1.ad.adatum.com** and the type **Host (A)**.
1. Click **Edit**.
1. In the Edit a DNS record, under **IP address**, type **10.1.1.9** and click **Save**.

#### PowerShell

Perform this task on CL1.

1. Open **Terminal**.
1. In Terminal, configure parameters:

    ```powershell
    $computerName = 'VN1-SRV5'
    $zoneName = 'ad.adatum.com'
    $oldDnsServerResourceRecord = Get-DnsServerResourceRecord `
        -ZoneName $zoneName `
        -RRType A `
        -Name 'vn1-srv1' `
        -ComputerName $computerName

    $newDnsServerResourceRecord = `
        [ciminstance]::new($oldDnsServerResourceRecord)
            
    $newDnsServerResourceRecord.RecordData.IPv4Address = '10.1.1.9'
    Set-DnsServerResourceRecord `
        -ZoneName $zoneName `
        -OldInputObject $oldDnsServerResourceRecord `
        -NewInputObject $newDnsServerResourceRecord `
        -ComputerName $computerName

### Task 4: Add the IP address of the decommissioned domain controller to the new domain controller

Perform this task on CL1.

1. Open **Terminal** as Administrator.
1. Create a CIM session to **VN1-SRV5**.

    ````powershell
    $cimSession = New-CimSession -ComputerName VN1-SRV5
    ````

1. Find the interface alias of subnet **10.1.1.0**.

    ````powershell
    $interfaceAlias = (
        Get-NetIPAddress -CimSession $cimSession -AddressFamily IPv4 |
        Where-Object { $PSItem.IPAddress -like '10.1.1.*' }
    ).InterfaceAlias
    ````

1. Add the IP address **10.1.1.8** with the prefix length of **24** to the interface.

    ````powershell
    New-NetIPAddress `
        -InterfaceAlias $interfaceAlias `
        -IPAddress 10.1.1.8 `
        -PrefixLength 24 `
        -CimSession $cimSession
    ````

1. Clear the DNS client cache on VN1-SRV5.

    ````powershell
    Clear-DnsClientCache -CimSession $cimSession
    ````

1. Remove the CIM session

    ````powershell
    Remove-CimSession $cimSession
    ````

### Task 5: Demote the old domain controller

#### Desktop experience

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, on the menu, click **Manage**, **Remove Roles and Features**.
1. In Remove Roles and Features Wizard, on page Befor You Begin, click **Next >**.
1. On page Server Selection, click **VN1-SRV1.ad.adatum.com** and click **Next >**.
1. On page Remove server roles, deactivate **Active Directory Domain Services**.
1. In dialog Remove features that require Active Directory Domain Services, click **Remove Features**.
1. In dialog Validation Results, click **Demote this domain controller**.
1. In Active Directory Domain Services Configuration Wizard, on page Credentials, click **Change...**.
1. In the dialog Credentials for deployment operation, enter the credentials for **Administrator@ad.adatum.com** and click **OK**.
1. On page **Credentials**, click **Next >**.
1. On page Warnings, activate **Proceed with removal** and click **Next >**.
1. On page Removal Options, deactivate **Remove DNS delegation** and click **Next >**.
1. On page New Administrator Password, in **Password** and **Confirm password**, type a secure password, and take a note.
1. On page Review Options, click **Demote**.
1. On page Results, click **Close**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Store the new local administrator password in a variable.

    ````powershell
    $localAdministratorPassword = Read-Host `
        -Prompt 'LocalAdministratorPassword' `
        -AsSecureString
    ````

1. At the prompt **LocalAdministratorPassword** enter a secure password and take a note.

1. Demote the domain controller VN1-SRV1.

    ````powershell
    $job = Invoke-Command -ComputerName VN1-SRV1 -AsJob -ScriptBlock {
        $localAdministratorPassword = ConvertTo-SecureString `
            -String $using:localAdministratorPassword -AsPlainText -Force
        Uninstall-ADDSDomainController `
            -LocalAdministratorPassword $localAdministratorPassword -Force
    }
    ````

1. Wait for the job to complete.

    ````powershell
    $job | Wait-Job
    ````

    This will take a few minutes.

1. Read the output of the job.

    ````powershell
    $job | Receive-Job
    ````

    The value of the property **Status** should be **Success**.

#### Troubleshooting

If you receive any error message while demoting the domain controller, perform the following troubleshooting steps on CL1.

1. Open **Terminal**.
1. Shut down **VN1-SRV1**.

    ````powershell
    Stop-Computer -ComputerName VN1-SRV1 -WsmanAuthentication Default
    ````

    From this point on, you should not start the computer again. Consider, deleting it from Hyper-V.

1. Start ntdsutil.exe.

    ````powershell
    ntdsutil.exe
    ````

1. Remove VN1-SRV1.ad.adatum.com as domain controller by executing the following commands.

    ````shell
    Metadata cleanup
    ````

1. In metadata cleanup, change to connections.

    ````shell
    Connections
    ````

1. In connections, connect to server VN1-SRV5.ad.adatum.com.

    ````shell
    Connect to server VN1-SRV5.ad.adatum.com
    ````

1. Quit connections.

    ````shell
    Quit
    ````

1. In metadata cleanup, enter operation target selection.

    ````shell
    Select operation target
    ````

1. In select operation target, list the domains.

    ````shell
    List domains
    ````

    Take a note of the number to the left of DC=ad, DC=adatum, DC=com.

1. Select the domain. Ensure, you use the number, you noted in the previous step.

    ````shell
    Select domain 0
    ````

1. List the sites.

    ````shell
    List sites
    ````

    Take a note of the number to the left of CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=ad,DC=adatum,DC=com.

1. Select the site. Ensure, you use the number, you noted in the previous step.

    ````shell
    Select site 0
    ````

1. List the servers for the domain in the site.

    ````shell
    List servers for domain in site
    ````

    Take a note of the number to the left of CN=VN1-SRV1,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=ad,DC=adatum,DC=com.

1. Select the server. Ensure, you use the number, you noted in the previous step.

    ````shell
    Select server 0
    ````

1. Quit the operation target selection.

    ````shell
    Quit
    ````

1. In metadata cleanup, remove the selected server.

    ````shell
    Remove selected server
    ````

1. In the message box Server Remove Confirmation Dialog, ensure, that you remove **CN=VN1-SRV1,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=ad,DC=adatum,DC=com**. Click **Yes**.

1. Quit metadata cleanup.

    ````shell
    Quit
    ````

1. Quit ntdsutil.exe.

    ````shell
    Quit
    ````

Leave out task 5 and skip to the next exercise.

### Task 6: Remove roles from the decommissioned domain controller

#### Desktop experience

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, on the menu, click **Manage**, **Remove Roles and Features**.
1. In Remove Roles and Features Wizard, on page Befor You Begin, click **Next >**.
1. On page Server Selection, click **VN1-SRV1.ad.adatum.com** and click **Next >**.
1. On page Remove server roles, deactivate **Active Directory Domain Services**.
1. In dialog Remove features that require Active Directory Domain Services, click **Remove Features**.
1. On page **Remove server roles**, deactivate **DNS Server**.
1. Expand **File and Storage Services**, **File and iSCSI Services**, and deactivate **File Server**.
1. Click **Next >**.
1. On page Remove features, expand **Remote Server Administration Tools**, **Role Administration Tools**, deactivate **AD DS and AD LDS Tools**, and click **Next >**.
1. On page Confirmation, click **Remove**.
1. On page Removal progress, wait for the removal to complete and click **Close**.
1. Click **All Servers**.
1. In the context-menu of **VN1-SRV1**, click **Remove Server**.
1. In the message box **Are you sure you want to remove these servers from Server Manager?**, click **OK**.
1. Open **Terminal**
1. Shut down **VN1-SRV1**.

    ````powershell
    Stop-Computer -ComputerName VN1-SRV1 -WsmanAuthentication Default
    ````

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Uninstall the features **Active Directory Domain Services**, **DNS** and **File Server** from **VN1-SRV1**.

    ````powershell
    Uninstall-WindowsFeature `
        -Name AD-Domain-Services, DNS, FS-FileServer `
        -ComputerName VN1-SRV1
    ````

1. Shut down **VN1-SRV1**.

    ````powershell
    Stop-Computer -ComputerName VN1-SRV1 -WsmanAuthentication Default
    ````

## Exercise 5: Raise the domain and forest functional level

1. Raise the domain functional level

1. Raise the forest functional level

### Task 1: Raise the domain functional level

#### Desktop experience

Perform this task on CL1.

1. Open **Active Directory Administrative Center**.
1. In Active Directory Administrative Center, in the context-menu of **ad (local)**, click **Raise the domain functional level...**

1. In Raise domain function level, click **OK**.

1. In the message box This change affects the entire domain. After you raise the domain functional level, it is possible that you may not be able to reverse it., click **OK**.

1. In the message box Raise Domain Functional Level, click **OK**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Set the domain mode to Windows Server 2016.

    ````powershell
    Set-ADDomainMode -Identity ad.adatum.com -DomainMode Windows2025Domain
    ````

    If you receive an error message **Set-ADDomainMode : A referral was returned from the server**, restart VN1-SRV5 and try again.

    ````powershell
    Restart-Computer -ComputerName vn1-srv5 -WsmanAuthentication Default
    ````

1. At the prompt **Performing the operation "Set" on target "DC=ad,DC=adatum,DC=com".**, enter **y**.

### Task 2: Raise the forest functional level

#### Destkop experience

Perform this task on CL1.

1. Open **Active Directory Administrative Center**.
1. In Active Directory Administrative Center, in the context-menu of **ad (local)**, click **Raise the forest functional level...**
1. In Raise forest function level, click **OK**.
1. In the message box This change affects the entire forest. After you raise the forest functional level, it is possible that you may not be able to reverse it., click **OK**
1. In Raise Forest Functional Level, click **OK**.

#### PowerShell

Perform this task on CL1.

1. In the context menu of **Start**, click **Terminal**.
1. Set the forest mode to Windows Server 2016.

    ````powershell
    Set-ADForestMode -Identity ad.adatum.com -ForestMode Windows2025Forest
    ````

1. At the prompt **Performing the operation "Set" on target "CN=Partitions,CN=Configuration,DC=ad,DC=adatum,DC=com".**, enter **y**.

1. Check the forest mode.

    ````powershell
    Get-ADForest
    ````

    > The value for ForestMode should be Windows2025Forest.

## Exercise 6: Enable database 32K pages

1. [Verify the that your have 32k page capable database](#task-1-verify-the-that-your-have-32k-page-capable-database)
1. [Enable the Database 32k pages optional feature](#task-2-enable-the-database-32k-pages-optional-feature)

### Task 1: Verify the that your have 32k page capable database

Perform this task on CL1.

1. Open **Terminal**.
1. In Terminal, verify the msDs-JetDBpageSize property.

    ````powershell
    Get-ADObject `
        -LDAPFilter '(ObjectClass=nTDSDSA)' `
        -SearchBase 'CN=Configuration,DC=ad,DC=adatum,DC=com' `
        -Properties msDS-JetDBPageSize |
    Format-List distinguishedName, msDs-JetDBPageSize
    ````

    msDs-JetDBPageSize should be 32768.

### Task 2: Enable the Database 32k pages optional feature

Perform this task on CL1.

1. Open **Terminal**.
1. In Terminal, enable the Database 32k pages optional feature.

    ````powershell
    Enable-ADOptionalFeature `
        -Identity 'Database 32k pages feature' `
        -Scope 'ForestOrConfigurationSet' `
        -Server VN1-SRV5.ad.adatum.com `
        -Target ad.adatum.com
    ````

1. At the prompt Confirm, enter **ye**.

## Exercise 7: Deploy a new forest

1. [Install Active Directory Domain Services on VN2-SRV2](#task-1-install-active-directory-domain-services-on-vn2-srv2)
1. [Configure Active Directory Domain Services as new forest](#task-2-configure-active-directory-domain-services-as-new-forest) with the name ad.contoso.com on VN2-SRV2.
1. [Change the DNS client settings](#task-3-change-the-dns-client-settings) on CL3 to use 10.1.2.16 (VN2-SRV2)
1. [Connect to domain](#task-4-connect-to-domain) ad.contoso.com on CL3.
1. [Configure forwarders](#task-5-configure-forwarders) on VN2-SRV2 to use 8.8.8.8 and 8.8.4.4.

### Task 1: Install Active Directory Domain Services on VN2-SRV2

#### Desktop experience

Perform this task on CL1.

1. Open **Server Manager**.
1. In Server Manager, in the menu, click **Manage**, **Add Roles and Features**.
1. In Add Roles and Features Wizard, on page Before You Begin, click **Next >**.
1. On page Installation Type, ensure **Role-based or feature-basedd installation** is selected and click **Next >**.
1. On page Server Selection, click **VN2-SRV2.ad.adatum.com**, and click **Next >**.
1. On page Server Roles, activate **Active Directory Domain Services**.
1. In the dialog **Add features that are required for Active Directory Domain Services?**, click **Add Features**
1. On page **Server Roles**, click **Next >**.
1. On page Features, click **Next >**.
1. On page **AD DS**, click **Next >**.
1. On page **Confirmation**, activate the checkbox **Restart the destination server automatically if required** and click **Install**.
1. On page **Results**, click **Close**.

#### Windows Admin Center

Perform this task on CL1.

1. Open **Microsoft Edge**.
1. In Microsoft Edge, navigate to <https://admincenter.ad.adatum.com/>
1. On the Windows Admin Center page, click **vn2-srv2.ad.adatum.com**.
1. On the page vn1-srv5.ad.adatum.com, unter Tools, click **Roles & features**.
1. Under Roles and features, click **Active Directory Domain Services** and click **Install**.
1. In the pane Install Role and Features, activate the checkbox **Reboot the server automatically if required**, and click **Yes**.

Wait, until you receive a notification about the successfully completed installation.

#### PowerShell

Peform this task on VN2-SRV2.

1. In the context menu of **Start**, click **Windows PowerShell (Admin)**.
1. Install the windows feature **Active Directory Domain Services**.

    ````powershell
    Install-WindowsFeature `
        -Name AD-Domain-Services -IncludeManagementTools -Restart
    ````

### Task 2: Configure Active Directory Domain Services as new forest

#### Desktop experience

Perform this task on VN2-SRV2.

1. Open **Server Manager**.
1. In Server Manager, click *Notifications* (the flag with the yellow warning triangle), and under the message **Configuration required for Active Directory Domain Services at VN2-SRV2**, click **Promote this server to a domain controller**.
1. In Active Directory Domain Services Configuration Wizard, on page Deployment Configuration, click **Add a new forest**. In **Root domain name**, type **ad.contoso.com** and click **Next >**.
1. On page **Domain Controller Options**, ensure **Domain Name System (DNS) server** and **Global Catalog (GC)** are activated. Under **Type the Directory Services Restore Mode (DSRM) password**, in **Password** and **Confirm password**, type a secure password and take a note. You will need the password for a later lab. Click **Next >**.
1. On page DNS Options, click **Next >**.
1. On page Additional Options, in **The NetBIOS domain name**, type **CONTOSO** and click **Next >**.
1. On page **Paths**, click **Next >**.

    Note: In real world, it is recommended to have the paths on a separate drive.

1. On page Review Options, click **Next >**.
1. On page Prerequisites Check, click **Install**.

Continue to the next task. The server will restart automatically.

#### PowerShell

Perform this task on VN2-SRV2.

1. In the context menu of **Start**, click **Windows PowerShell (Admin)**.
1. Store the Directory Services Restore Mode (DSRM) password in a variable.

    ````powershell
    $safeModeAdministratorPassword = Read-Host `
        -Prompt 'Directory Services Restore Mode (DSRM) password' `
        -AsSecureString
    ````

1. At the prompt **Directory Services Restore Mode (DSRM) password** enter a secure password and take a note.
1. Install a new forest with the domain name **ad.contoso.com** and the NetBIOS name **CONTOSO**.

    ````powershell
    Install-ADDSForest `
        -DomainName ad.contoso.com `
        -DomainNetbiosName CONTOSO `
        -SafeModeAdministratorPassword $safeModeAdministratorPassword
    ````

1. At the prompt **The target server will be configured as a domain controller and restarted when this operation is complete.** type **y**.

Continue to the next task. The server will restart automatically.

### Task 3: Change the DNS client settings

#### Desktop experience

Perform this task on CL3.

1. Open **Settings**.
1. In Settings, in the left pane, click **Network & internet**.
1. In Network & internet, click **Ethernet**.
1. In Ethernet, beside **DNS server assignment**, click **Edit**.
1. In Edit IP Settings, under **Preferred DNS**, type **10.1.2.16** and click **Save**.
1. Sign out.

#### PowerShell

Perform this task on CL3.

1. In the context menu of **Start**, click **Terminal (Admin)**.
1. Set the DNS client server address on the interface **Ethernet** to **10.1.2.16**.

    ````powershell
    Set-DnsClientServerAddress -InterfaceAlias Ethernet -ServerAddresses 10.1.2.16
    ````

### Task 4: Connect to domain

Note: Wait for VN2-SRV2 to reboot before starting with this task.

#### Desktop experience

Perform this task on CL3.

1. Open **Settings**.
1. In Settings, in the left pane, click **Accounts**.
1. In Accounts, click **Access work or school**.
1. In Access work or school, beside **Add a work or school account**, click **Connect**.
1. In Set up a work or school account, click the link **Join this device to a local Active Directory domain**.
1. In Join a domain, under **Domain name**, type **ad.contoso.com** and click **Next**.
1. In Windows Security, enter the credentials for **Administrator@ad.contoso.com**.
1. In Add an account, click **Skip**.
1. In Restart your PC, click **Restart now**.

#### PowerShell

Perform this task on CL3.

1. In the context menu of **Start**, click **Terminal (Admin)**.
1. Add the computer to the domain **ad.contoso.com** and restart it.

    ````powershell
    Add-Computer -DomainName ad.contoso.com -Restart
    ````

1. In **Windows PowerShell credential request**, enter the the credentials of **Administrator@ad.contoso.com**.

### Task 5: Configure forwarders

#### Desktop experience

Perform this task on VN2-SRV2.

1. Sign in as **Administrator@ad.contoso.com**.
1. Open **DNS**.
1. In DNS Manager, click **VN2-SRV2**.
1. In vn2-srv2, double-click **Forwarders**.
1. In vn2-srv2 Properties, on tab Forwarders, click **Edit...**
1. In Edit Forwarders, click **10.1.1.8** and click **Delete**.
1. In **\<Click here to add an IP Address or DNS Name\>**, enter **8.8.8.8**. Repeat this step with **8.8.4.4** and click **OK**.
1. In **vn2-srv2 Properties**, click **OK**.

#### PowerShell

Perform this task on VN2-SRV2.

1. Sign in as **Administrator@ad.contoso.com**.
1. Run **Windows PowerShell (Admin)**.
1. In Windows PowerShell (Admin), configure the forwarder to **8.8.8.8** and **8.8.4.4**.

    ````powershell
    Set-DnsServerForwarder -IPAddress 8.8.8.8, 8.8.4.4
    ````

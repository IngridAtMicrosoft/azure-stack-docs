---
title: How to establish a VNET to VNET connection in Azure Stack Hub with Fortinet FortiGate NVA 
description: Learn how to establish a VNET to VNET connection in Azure Stack Hub with Fortinet FortiGate NVA
author: mattbriggs

ms.topic: how-to
ms.date: 1/22/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 10/03/2019

# Intent: Notdone: As a < type of user >, I want < what? > so that < why? >
# Keyword: Notdone: keyword noun phrase

---


# Establish a VNET to VNET connection in Azure Stack Hub with Fortinet FortiGate NVA

In this article, you'll connect a VNET in one Azure Stack Hub to a VNET in another Azure Stack Hub using Fortinet FortiGate NVA, a network virtual appliance.

This article addresses the current Azure Stack Hub limitation, which lets tenants to only set up one VPN connection across two environments. Users will learn how to set up a custom gateway on a Linux virtual machine that will allow multiple VPN connections across different Azure Stack Hub. The procedure in this article deploys two VNETs with a FortiGate NVA in each VNET: one deployment per Azure Stack Hub environment. It also details the changes required to set up an IPSec VPN between the two VNETs. The steps in this article should be repeated for each VNET in each Azure Stack Hub. 

## Prerequisites

-  Access to an Azure Stack Hub integrated systems with available capacity to deploy the required compute, network, and resource requirements needed for this solution. 

    > [!Note]  
    > These instructions will **not** work with an Azure Stack Development Kit (ASDK) because of the network limitations in the ASDK. For more information, see [ASDK requirements and considerations](https://docs.microsoft.com/azure-stack/asdk/asdk-deploy-considerations).

-  A network virtual appliance (NVA) solution downloaded and published to the Azure Stack Hub Marketplace. An NVA controls the flow of network traffic from a perimeter network to other networks or subnets. This procedure uses the [Fortinet FortiGate Next-Generation Firewall Single VM Solution](https://azuremarketplace.microsoft.com/marketplace/apps/fortinet.fortinet-FortiGate-singlevm).

-  At least two available FortiGate license files to activate the FortiGate NVA. Information on how to get these licenses, see the Fortinet Document Library article [Registering and downloading your license](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license).

    This procedure uses the [Single FortiGate-VM deployment](ttps://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/632940/single-FortiGate-vm-deployment). You can find steps on how to connect the FortiGate NVA to the Azure Stack Hub VNET to in your on-premises network.

    For more information on how to deploy the FortiGate solution in an active-passive (HA) set up, see the Fortinet Document Library article [HA for FortiGate-VM on Azure](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/983245/ha-for-FortiGate-vm-on-azure).

## Deployment parameters

The following table summarizes the parameters that are used in these deployments for reference:

### Deployment one: Forti1

| FortiGate Instance Name | Forti1 |
|-----------------------------------|---------------------------|
| BYOL License/Version | 6.0.3 |
| FortiGate administrative username | fortiadmin |
| Resource Group name | forti1-rg1 |
| Virtual network name | forti1vnet1 |
| VNET Address Space | 172.16.0.0/16* |
| Public VNET subnet name | forti1-PublicFacingSubnet |
| Public VNET address prefix | 172.16.0.0/24* |
| Inside VNET subnet name | forti1-InsideSubnet |
| Inside VNET subnet prefix | 172.16.1.0/24* |
| VM Size of FortiGate NVA | Standard F2s_v2 |
| Public IP address name | forti1-publicip1 |
| Public IP address type | Static |

### Deployment two: Forti2

| FortiGate Instance Name | Forti2 |
|-----------------------------------|---------------------------|
| BYOL License/Version | 6.0.3 |
| FortiGate administrative username | fortiadmin |
| Resource Group name | forti2-rg1 |
| Virtual network name | forti2vnet1 |
| VNET Address Space | 172.17.0.0/16* |
| Public VNET subnet name | forti2-PublicFacingSubnet |
| Public VNET address prefix | 172.17.0.0/24* |
| Inside VNET subnet name | Forti2-InsideSubnet |
| Inside VNET subnet prefix | 172.17.1.0/24* |
| VM Size of FortiGate NVA | Standard F2s_v2 |
| Public IP address name | Forti2-publicip1 |
| Public IP address type | Static |

> [!Note]
> \* Choose a different set of address spaces and subnet prefixes if the above overlap in any way with the on-premises network environment including the VIP Pool of either Azure Stack Hub. Also ensure that the address ranges do not overlap with one another.**

## Deploy the FortiGate NGFW Marketplace Items

Repeat these steps for both Azure Stack Hub environments. 

1. Open the Azure Stack Hub user portal. Be sure to use credentials that have at least Contributor rights to a subscription.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image5.png)

1. Select **Create a resource** and search for `FortiGate`.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image6.png)

2. Select the **FortiGate NGFW** and select the **Create**.

3. Complete **Basics** using the parameters from the [Deployment parameters](#deployment-parameters) table.

    Your form should contain the following information:

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image7.png)

4. Select **OK**.

5. Provide the virtual network, subnets, and VM size details from the [Deployment parameters](#deployment-parameters).

    If you wish to use different names and ranges, take care not to  use parameters that will conflict with the other VNET and FortiGate resources in the other Azure Stack Hub environment. This is especially true when setting the VNET IP range and subnet ranges within the VNET. Check that they don't overlap with the IP ranges for the other VNET you create.

6. Select **OK**.

7. Configure the public IP that will be used for the FortiGate NVA:

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image8.png)

8. Select **OK** and then Select **OK**.

9. Select **Create**.

The deployment will take about 10 minutes. You can now repeat the steps to create the other FortiGate NVA and VNET deployment in the other Azure Stack Hub environment.

## Configure routes (UDRs) for each VNET

Perform these steps for both deployments, forti1-rg1 and forti2-rg1.

1. Navigate to the forti1-rg1 Resource Group in the Azure Stack Hub portal.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image9.png)

2. Select on the 'forti1-forti1-InsideSubnet-routes-xxxx' resource.

3. Select **Routes** under **Settings**.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image10.png)

4. Delete the **to-Internet** Route.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image11.png)

5. Select **Yes**.

6. Select **Add**.

7. Name the **Route** `to-forti1` or `to-forti2`. Use your IP range if you are using a different IP range.

8. Enter:
    - forti1: `172.17.0.0/16`  
    - forti2: `172.16.0.0/16`  

    Use your IP range if you are using a different IP range.

9. Select **Virtual appliance** for the **Next hop type**.
    - forti1: `172.16.1.4`  
    - forti2: `172.17.0.4`  

    Use your IP range if you are using a different IP range.

    ![](./media/azure-stack-network-howto-vnet-to-vnet-stacks/image12.png)

10. Select **Save**.

Repeat the steps for each **InsideSubnet** route for each resource group.

## Activate the FortiGate NVAs and Configure an IPSec VPN connection on each NVA

 You will require a valid license file from Fortinet to activate each FortiGate NVA. The NVAs will **not** function until you have activated each NVA. For more information how to get a license file and steps to activate the NVA, see the Fortinet Document Library article [Registering and downloading your license](https://docs2.fortinet.com/vm/azure/FortiGate/6.2/azure-cookbook/6.2.0/19071/registering-and-downloading-your-license).

Two license files will need to be acquired – one for each NVA.

## Create an IPSec VPN between the two NVAs

Once the NVAs have been activated, follow these steps to create an IPSec VPN between the two NVAs.

Following the below steps for both the forti1 NVA and forti2 NVA:

1.  Get the assigned Public IP address by navigating to the fortiX VM Overview page:

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image13.png)

2.  Copy the assigned IP address, open a browser, and paste the address into the address bar. Your browser may warn you that the security certificate is not trusted. Continue anyway.

4.  Enter the FortiGate administrative user name and password you provided during the deployment.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image14.png)

5.  Select **System** > **Firmware**.

6.  Select the box showing the latest firmware, for example, `FortiOS v6.2.0 build0866`.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image15.png)

7.  Select **Backup config and upgrade** and Continue when prompted.

8.  The NVA updates its firmware to the latest build and reboots. The process takes about five minutes. Log back into the FortiGate web console.

10.  Click **VPN** > **IPSec Wizard**.

11. Enter a name for the VPN, for example, `conn1` in the **VPN Creation Wizard**.

12. Select **This site is behind NAT**.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image16.png)

13. Select **Next**.

14. Enter the remote IP address of the on-premises VPN device to which you are going to connect.

15. Select **port1** as the **Outgoing Interface**.

16. Select **Pre-shared Key** and enter (and record) a pre-shared key. 

    > [!Note]  
    > You will need this key to set up the connection on the on-premises VPN device, that is, they must match *exactly*.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image17.png)

17. Select **Next**.

18. Select **port2** for the **Local Interface**.

19. Enter the local subnet range:
    - forti1: 172.16.0.0/16
    - forti2: 172.17.0.0/16

    Use your IP range if you are using a different IP range.

20. Enter the appropriate Remote Subnet(s) that represent the on-premises network, which you will connect to through the on-premises VPN device.
    - forti1: 172.16.0.0/16
    - forti2: 172.17.0.0/16

    Use your IP range if you are using a different IP range.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image18.png)

21. Select **Create**

22. Select **Network** > **Interfaces**.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image19.png)

23. Double-click **port2**.

24. Choose **LAN** in the **Role** list and **DHCP** for the Addressing mode.

25. Select **OK**.

Repeat the steps for the other NVA.


## Bring Up All Phase 2 Selectors 

Once the above has been completed for **both** NVAs:

1.  On the forti2 FortiGate web console, select to **Monitor** > **IPsec Monitor**. 

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image20.png)

2.  Highlight `conn1` and select the **Bring Up** > **All Phase 2 Selectors**.

    ![](./media/azure-stack-network-howto-vnet-to-vnet/image21.png)


## Test and validate connectivity

You should now be able to route in between each VNET via the FortiGate NVAs. To validate the connection, create an Azure Stack Hub VM in each VNET's InsideSubnet. Creating an Azure Stack Hub VM can be done via the portal, CLI, or PowerShell. When creating the VMs:

-   The Azure Stack Hub VMs are placed on the **InsideSubnet** of each VNET.

-   You do **not** apply any NSGs to the VM upon creation (That is, remove the NSG that gets added by default if creating the VM from the portal.

-   Ensure that the VM firewall rules allow the communication you are going to use to test connectivity. For testing purposes, it is recommended to disable the firewall completely within the OS if at all possible.

## Next steps

[Differences and considerations for Azure Stack Hub networking](azure-stack-network-differences.md)  
[Offer a network solution in Azure Stack Hub with Fortinet FortiGate](../operator/azure-stack-network-solutions-enable.md)  
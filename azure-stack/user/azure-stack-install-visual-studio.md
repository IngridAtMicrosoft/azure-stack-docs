---
title: Install Visual Studio and connect to Azure Stack Hub 
description: Learn how to install Visual Studio and connect to Azure Stack Hub.
author: sethmanheim

ms.topic: article
ms.date: 01/07/2020
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 01/04/2020

# Intent: As an Azure Stack user, I want to install Visual Studio on Azure stack so I can write and deploy Azure Resource Manager templates.
# Keyword: install visual studio azure stack

---


# Install Visual Studio and connect to Azure Stack Hub

You can use Visual Studio to write and deploy Azure Resource Manager [templates](azure-stack-arm-templates.md) to Azure Stack Hub. The steps in this article describe how to install Visual Studio on [Azure Stack Hub](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp) or on an external computer if you plan to use Azure Stack Hub through [VPN](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn).

## Install Visual Studio

1. Download and run the [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx).  

2. Open the **Microsoft Web Platform Installer**.

3. Search for **Visual Studio Community 2015 with Microsoft Azure SDK - 2.9.6**. Click **Add**, and then **Install**.

4. Uninstall the **Microsoft Azure PowerShell** that's installed as part of the Azure SDK.

    ![Screenshot of WebPI install steps](./media/azure-stack-install-visual-studio/image1.png)

5. [Install PowerShell for Azure Stack Hub](../operator/azure-stack-powershell-install.md).

6. Restart the operating system after the installation completes.

## Connect to Azure Stack Hub with Azure AD

1. Launch Visual Studio.

2. From the **View** menu, select **Cloud Explorer**.

3. In the new pane, select **Add Account** and sign in with your Azure Active Directory (Azure AD) credentials.  

    ![Screenshot of Cloud Explorer once logged in and connected to Azure Stack Hub](./media/azure-stack-install-visual-studio/image2.png)

Once logged in, you can [deploy templates](azure-stack-deploy-template-visual-studio.md) or browse available resource types and resource groups to create your own templates.  

## Connect to Azure Stack Hub with AD FS

1. Launch Visual Studio.

2. From **Tools**, select **Options**.

3. Expand **Environment** in the **Navigation Pane** and select **Accounts**.

4. Select **Add**, and enter the User Azure Resource Manger endpoint. For the Azure Stack Development Kit (ASDK), the URL is: `https://management.local.azurestack/external`.  For Azure Stack Hub integrated systems, the URL is: `https://management.[Region}.[External FQDN]`.

    ![Add new Azure Cloud discovery endpoint](./media/azure-stack-install-visual-studio/image5.png)

5. Select **Add**.  

    Visual Studio calls Azure Resource Manger and discovers the endpoints, including the authentication endpoint, for Azure Directory Federated Services (AD FS).

    ![Screenshot of Cloud Explorer once logged in and connected to Azure Stack Hub](./media/azure-stack-install-visual-studio/image6.png)

6. Select **Cloud Explorer** from the **View** menu.

7. Select **Add Account** and sign in with your AD FS credentials.  

    ![Sign in to Visual Studio in Cloud Explorer](./media/azure-stack-install-visual-studio/image7.png)

    Cloud Explorer queries the available subscriptions. You can select an available subscription to manage.

    ![Select subscriptions to manage in Cloud Explorer](./media/azure-stack-install-visual-studio/image8.png)

8. Browse your existing resources, resource groups, or deploy templates.

## Next steps

- Read more about Visual Studio [side by side](/visualstudio/install/install-visual-studio-versions-side-by-side) with other Visual Studio versions.
- [Develop templates for Azure Stack Hub](azure-stack-develop-templates.md).

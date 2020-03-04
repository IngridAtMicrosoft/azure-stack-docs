---
title: Remove the MySQL resource provider in Azure Stack Hub 
description: Learn how to remove the MySQL resource provider from your Azure Stack Hub deployment.
author: bryanla
ms.topic: article
ms.date: 1/22/2020
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 11/20/2

# Intent: As an Azure Stack operator, I want to remove the MySQL resource provider on Azure Stack.
# Keyword: azure stack remove mysql resource provider

---


# Remove the MySQL resource provider in Azure Stack Hub

Before you remove the MySQL resource provider, you must remove all the provider dependencies. You'll also need a copy of the deployment package that was used to install the resource provider.

> [!NOTE]
> You can find the download links for the resource provider installers in [Deploy the resource provider prerequisites](./azure-stack-mysql-resource-provider-deploy.md#prerequisites).

Removing the MySQL resource provider will delete the associated plans and quotas managed by operator. But it doesn't delete tenant databases from hosting servers.

## To remove the MySQL resource provider

1. Verify that you've removed all the existing MySQL resource provider dependencies.

   > [!NOTE]
   > Uninstalling the MySQL resource provider will proceed even if dependent resources are currently using the resource provider.
  
2. Get a copy of the MySQL resource provider installation package and then run the self-extractor to extract the contents to a temporary directory.
3. Open a new elevated PowerShell console window and change to the directory where you extracted the MySQL resource provider installation files.
4. Run the DeployMySqlProvider.ps1 script using the following parameters:
    - **Uninstall**: Removes the resource provider and all associated resources.
    - **PrivilegedEndpoint**: The IP address or DNS name of the privileged endpoint.
    - **AzureEnvironment**: The Azure environment used for deploying Azure Stack Hub. Required only for Azure AD deployments.
    - **CloudAdminCredential**: The credential for the cloud administrator, necessary to access the privileged endpoint.
    - **DirectoryTenantID**
    - **AzCredential**: The credential for the Azure Stack Hub service admin account. Use the same credentials that you used for deploying Azure Stack Hub.

## Next steps

[Offer App Services as PaaS](azure-stack-app-service-overview.md)

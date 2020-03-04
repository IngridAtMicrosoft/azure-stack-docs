---
title: Update the Azure Stack Hub SQL resource provider
titleSuffix: Azure Stack Hub
description: Learn how to update the Azure Stack Hub SQL resource provider.
author: bryanla

ms.topic: article
ms.date: 11/11/2019
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 11/11/2019

# Intent: As an Azure Stack operator, I want to update the SQL resource provider.
# Keyword: update sql resource provider azure stack

---


# Update the SQL resource provider

A new SQL resource provider might be released when Azure Stack Hub is updated to a new build. Although the existing resource provider continues to work, we recommend updating to the latest build as soon as possible.

Starting with the SQL resource provider version 1.1.33.0 release, updates are cumulative and don't need to be installed in the order in which they were released as long as you're starting from version 1.1.24.0 or later. For example, if you're running version 1.1.24.0 of the SQL resource provider, then you can upgrade to version 1.1.33.0 or later without needing to first install version 1.1.30.0. To review available resource provider versions, and the version of Azure Stack Hub they're supported on, see the versions list in [Deploy the resource provider prerequisites](./azure-stack-sql-resource-provider-deploy.md#prerequisites).

To update the resource provider, use the *UpdateSQLProvider.ps1* script. Use your service account with local administrative rights and is an **owner** of the subscription. This script is included with the download of the new SQL resource provider. The update process is similar to the process used to [Deploy the resource provider](./azure-stack-sql-resource-provider-deploy.md). The update script uses the same arguments as the DeploySqlProvider.ps1 script, and you'll need to provide certificate information.

 > [!IMPORTANT]
 > Before upgrading the resource provider, review the release notes to learn about new functionality, fixes, and any known issues that could affect your deployment.

## Update script processes

The *UpdateSQLProvider.ps1* script creates a new virtual machine (VM) with the latest resource provider code.

> [!NOTE]
> We recommend that you download the latest Windows Server 2016 Core image from Marketplace Management. If you need to install an update, you can place a **single** MSU package in the local dependency path. The script will fail if there's more than one MSU file in this location.

After the *UpdateSQLProvider.ps1* script creates a new VM, the script migrates the following settings from the old provider VM:

* database information
* hosting server information
* required DNS record

## Update script parameters

You can specify the following parameters from the command line when you run the **UpdateSQLProvider.ps1** PowerShell script. If you don't, or if any parameter validation fails, you're prompted to provide the required parameters.

| Parameter name | Description | Comment or default value |
| --- | --- | --- |
| **CloudAdminCredential** | The credential for the cloud admin, necessary for accessing the privileged endpoint. | _Required_ |
| **AzCredential** | The credentials for the Azure Stack Hub service admin account. Use the same credentials that you used for deploying Azure Stack Hub. | _Required_ |
| **VMLocalCredential** | The credentials for the local admin account of the SQL resource provider VM. | _Required_ |
| **PrivilegedEndpoint** | The IP address or DNS name of the privileged endpoint. |  _Required_ |
| **AzureEnvironment** | The Azure environment of the service admin account which you used for deploying Azure Stack Hub. Required only for Azure AD deployments. Supported environment names are **AzureCloud**, **AzureUSGovernment**, or if using a China Azure AD, **AzureChinaCloud**. | AzureCloud |
| **DependencyFilesLocalPath** | You must also put your certificate .pfx file in this directory. | _Optional for single node, but mandatory for multi-node_ |
| **DefaultSSLCertificatePassword** | The password for the .pfx certificate. | _Required_ |
| **MaxRetryCount** | The number of times you want to retry each operation if there's a failure.| 2 |
| **RetryDuration** |The timeout interval between retries, in seconds. | 120 |
| **Uninstall** | Removes the resource provider and all associated resources. | No |
| **DebugMode** | Prevents automatic cleanup on failure. | No |

## Update script PowerShell example
> [!NOTE]
> This update process only applies to Azure Stack Hub integrated systems.

If you're updating the SQL resource provider version to 1.1.33.0 or previous versions, you need to install specific versions of AzureRm.BootStrapper and Azure Stack Hub modules in PowerShell. If you're updating to the SQL resource provider version 1.1.47.0, the deployment script will automatically download and install the necessary PowerShell modules for you to path C:\Program Files\SqlMySqlPsh.

```powershell
# Install the AzureRM.Bootstrapper module, set the profile, and install the AzureStack module.
# Note that this might not be the most currently available version of Azure Stack Hub PowerShell.
Install-Module -Name AzureRm.BootStrapper -Force
Use-AzureRmProfile -Profile 2018-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.6.0
```

> [!NOTE]
> In disconnected scenario, you need to download the required PowerShell modules and register the repository manually as a prerequisite. You can get more information in [Deploy SQL resource provider](azure-stack-sql-resource-provider-deploy.md)

The following is an example of using the *UpdateSQLProvider.ps1* script that you can run from an elevated PowerShell console. Be sure to change the variable information and passwords as needed:  

```powershell
# Use the NetBIOS name for the Azure Stack Hub domain. On the Azure Stack Hub SDK, the default is AzureStack but this might have been changed at installation.
$domain = "AzureStack"

# For integrated systems, use the IP address of one of the ERCS VMs.
$privilegedEndpoint = "AzS-ERCS01"

# Provide the Azure environment used for deploying Azure Stack Hub. Required only for Azure AD deployments. Supported values for the <environment name> parameter are AzureCloud, AzureChinaCloud, or AzureUSGovernment depending which Azure subscription you're using.
$AzureEnvironment = "<EnvironmentName>"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\SQLRP'

# The service admin account (this can be Azure AD or AD FS).
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set the credentials for the new resource provider VM.
$vmLocalAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("sqlrpadmin", $vmLocalAdminPass)

# Add the cloudadmin credential required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force

# For version 1.1.47.0, the PowerShell modules used by the RP deployment are placed in C:\Program Files\SqlMySqlPsh
# The deployment script adds this path to the system $env:PSModulePath to ensure correct modules are used.
$rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
$env:PSModulePath = $env:PSModulePath + ";" + $rpModulePath

# Change directory to the folder where you extracted the installation files.
# Then adjust the endpoints.
. $tempDir\UpdateSQLProvider.ps1 -AzCredential $AdminCreds `
  -VMLocalCredential $vmLocalAdminCreds `
  -CloudAdminCredential $cloudAdminCreds `
  -PrivilegedEndpoint $privilegedEndpoint `
  -AzureEnvironment $AzureEnvironment `
  -DefaultSSLCertificatePassword $PfxPass `
  -DependencyFilesLocalPath $tempDir\cert

 ```

When the resource provider update script finishes, close the current PowerShell session.

## Next steps

[Maintain the SQL resource provider](azure-stack-sql-resource-provider-maintain.md)

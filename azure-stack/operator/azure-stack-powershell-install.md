---
title: Install PowerShell for Azure Stack Hub 
description: Learn how to install PowerShell for Azure Stack Hub.
author: mattbriggs

ms.topic: article
ms.date: 1/22/2020
ms.author: mabrigg
ms.reviewer: thoroet
ms.lastreviewed: 09/19/2019

# Intent: As an Azure Stack operator, I want to install Powershell for Azure Stack.
# Keyword: install powershell azure stack

---

# Install PowerShell for Azure Stack Hub

Azure PowerShell provides a set of cmdlets that use the Azure Resource Manager model for managing your Azure Stack Hub resources.

To work with your cloud, you need to install Azure Stack Hub compatible PowerShell modules. Azure Stack Hub uses the **AzureRM** module instead of the newer **AzureAZ** module used in global Azure. You also need to use *API profiles* to specify the compatible endpoints for the Azure Stack Hub resource providers.

API profiles provide a way to manage version differences between Azure and Azure Stack Hub. An API version profile is a set of Azure Resource Manager PowerShell modules with specific API versions. Each cloud platform has a set of supported API version profiles. For example, Azure Stack Hub supports a specific profile version such as **2019-03-01-hybrid**. When you install a profile, the Azure Resource Manager PowerShell modules that correspond to the specified profile are installed.

You can install Azure Stack Hub compatible PowerShell modules in internet-connected, partially connected, or disconnected scenarios. This article walks you through the detailed instructions for these scenarios.

## 1. Verify your prerequisites

Before you get started with Azure Stack Hub and PowerShell, you must have the following prerequisites:

- **PowerShell Version 5.0** <br>
To check your version, run **$PSVersionTable.PSVersion** and compare the **Major** version. If you don't have PowerShell 5.0, follow the [Installing Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell).

  > [!Note]
  > PowerShell 5.0 requires a Windows machine.

- **Run Powershell in an elevated command prompt**.

- **PowerShell Gallery access** <br>
  You need access to the [PowerShell Gallery](https://www.powershellgallery.com). The gallery is the central repository for PowerShell content. The **PowerShellGet** module contains cmdlets for discovering, installing, updating, and publishing PowerShell artifacts. Examples of these artifacts are modules, DSC resources, role capabilities, and scripts from the PowerShell Gallery and other private repositories. If you're using PowerShell in a disconnected scenario, you must retrieve resources from a machine with a connection to the internet and store them in a location accessible to your disconnected machine.

## 2. Validate the PowerShell Gallery accessibility

Validate if PSGallery is registered as a repository.

> [!Note]  
> This step requires internet access.

Open an elevated PowerShell prompt, and run the following cmdlets:

```powershell
Import-Module -Name PowerShellGet -ErrorAction Stop
Import-Module -Name PackageManagement -ErrorAction Stop
Get-PSRepository -Name "PSGallery"
```

If the repository isn't registered, open an elevated PowerShell session and run the following command:

```powershell
Register-PSRepository -Default
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
```

## 3. Uninstall existing versions of the Azure Stack Hub PowerShell modules

Before installing the required version, make sure that you uninstall any previously installed Azure Stack Hub AzureRM PowerShell modules. Uninstall the modules by using one of the following two methods:

1. To uninstall the existing AzureRM PowerShell modules, close all the active PowerShell sessions, and run the following cmdlets:

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose
    ```

    If you hit an error such as 'The module is already in use', close the PowerShell sessions that are using the modules and rerun the above script.

2. Delete all the folders that start with `Azure` or `Azs.` from the `C:\Program Files\WindowsPowerShell\Modules` and `C:\Users\{yourusername}\Documents\WindowsPowerShell\Modules` folders. Deleting these folders removes any existing PowerShell modules.

## 4. Connected: Install PowerShell for Azure Stack Hub with internet connectivity

The API version profile and Azure Stack Hub PowerShell modules you require will depend on the version of Azure Stack Hub you're running.

### Install Azure Stack Hub PowerShell

Run the following PowerShell script to install these modules on your development workstation:

- For Azure Stack Hub 1910 or later:

    ```powershell  
    # Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
    Install-Module -Name AzureRM.BootStrapper

    # Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.
    Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 1.8.0
    ```

- For Azure Stack Hub 1908 or after 1903:

    ```powershell  
    # Install the AzureRM.BootStrapper module. Select Yes when prompted to install NuGet
    Install-Module -Name AzureRM.BootStrapper

    # Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.
    Use-AzureRmProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 1.7.2
    ```
  
- For Azure Stack Hub version 1903 or earlier, only install the two modules below:

    ```powershell  
    # Install and import the API Version Profile required by Azure Stack Hub into the current PowerShell session.

    Install-Module -Name AzureRM -RequiredVersion 2.4.0
    Install-Module -Name AzureStack -RequiredVersion 1.7.1
    ```

    > [!Note]  
    > - Azure Stack Hub module version 1.8.0 is a breaking change release. Refer to the [release note](release-notes.md) for details.
    > - The Azure Stack Hub module version 1.7.2 is a breaking change release. To migrate from Azure Stack Hub 1.6.0, please refer to the [migration guide](https://aka.ms/azspshmigration171).
    > - The AzureRM module version 2.4.0 comes with a breaking change for the cmdlet Remove-AzureRmStorageAccount. This cmdlet expects `-Force` parameter to be specified for removing the storage account without confirmation.
    > - You don't need to install **AzureRM.BootStrapper** to install the modules for Azure Stack Hub version 1901 or later.
    > - Don't install the 2018-03-01-hybrid profile in addition to using the above AzureRM modules on Azure Stack Hub version 1901 or later.

### Confirm the installation of PowerShell

Confirm the installation by running the following command:

```powershell
Get-Module -Name "Azure*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

If the installation is successful, the `AzureRM` and `AzureStack` modules are displayed in the output.

## 5. Disconnected: Install PowerShell without an internet connection

In a disconnected scenario, you first download the PowerShell modules to a machine that has internet connectivity. Then, you transfer them to the Azure Stack Development Kit (ASDK) for installation.

Sign in to a computer with internet connectivity and use the following scripts to download the Azure Resource Manager and Azure Stack Hub packages, depending on your version of Azure Stack Hub.

Installation has four steps:

1. Install Azure Stack Hub PowerShell to a connected machine.
2. Enable additional storage features.
3. Transport the PowerShell packages to your disconnected workstation.
4. Manually bootstrap the NuGet provider on your disconnected workstation.
5. Confirm the installation of PowerShell.

### Install Azure Stack Hub PowerShell

- Azure Stack Hub 1910 or later.

    ```powershell
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop

    $Path = "<Path that is used to save the packages>"
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.8.0
    ```

- For Azure Stack Hub 1908 or after 1903:

    ```powershell
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop

    $Path = "<Path that is used to save the packages>"
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.7.2
    ```

- Azure Stack Hub 1903 or earlier.

    ```powershell
    Import-Module -Name PowerShellGet -ErrorAction Stop
    Import-Module -Name PackageManagement -ErrorAction Stop

    $Path = "<Path that is used to save the packages>"
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.4.0
    Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.7.1
    ```

    > [!Note]  
    > - Azure Stack Hub module version 1.8.0 is a breaking change release. Refer to the [release note](release-notes.md) for details.
    > The Azure Stack Hub module version 1.7.1 is a breaking change. To migrate from Azure Stack Hub 1.6.0 please refer to the [migration guide](https://github.com/Azure/azure-powershell/tree/AzureRM/documentation/migration-guides/Stack).

    > [!NOTE]
    > On machines without an internet connection, we recommend executing the following cmdlet for disabling the telemetry data collection. You may experience a performance degradation of the cmdlets without disabling the telemetry data collection. This is applicable only for the machines without internet connections
    > ```powershell
    > Disable-AzureRmDataCollection
    > ```

### Add your packages to your workstation

1. Copy the downloaded packages to a USB device.

2. Sign in to the disconnected workstation and copy the packages from the USB device to a location on the workstation.

3. Manually bootstrap the NuGet provider on your disconnected workstation. For instructions, see [Manually bootstrapping the NuGet provider on a machine that isn't connected to the internet](https://docs.microsoft.com/powershell/scripting/gallery/how-to/getting-support/bootstrapping-nuget#manually-bootstrapping-the-nuget-provider-on-a-machine-that-is-not-connected-to-the-internet).

4. Register this location as the default repository and install the AzureRM and `AzureStack` modules from this repository:

   ```powershell
   # requires -Version 5
   # requires -RunAsAdministrator
   # requires -Module PowerShellGet
   # requires -Module PackageManagement

   $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
   $RepoName = "MyNuGetSource"

   Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

   Install-Module -Name AzureRM -Repository $RepoName

   Install-Module -Name AzureStack -Repository $RepoName
   ```

### Confirm the installation of PowerShell

Confirm the installation by running the following command:

```powershell
Get-Module -Name "Azure*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

## 6. Configure PowerShell to use a proxy server

In scenarios that require a proxy server to access the internet, you first configure PowerShell to use an existing proxy server:

1. Open an elevated PowerShell prompt.
2. Run the following commands:

   ```powershell
   #To use Windows credentials for proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials

   #Alternatively, to prompt for separate credentials that can be used for #proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = Get-Credential
   ```

## Next steps

- [Download Azure Stack Hub tools from GitHub](azure-stack-powershell-download.md)
- [Configure the Azure Stack Hub user's PowerShell environment](../user/azure-stack-powershell-configure-user.md)
- [Configure the Azure Stack Hub operator's PowerShell environment](azure-stack-powershell-configure-admin.md)
- [Manage API version profiles in Azure Stack Hub](../user/azure-stack-version-profiles.md)

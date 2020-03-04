---
title: Azure Stack Hub servicing policy
titleSuffix: Azure Stack Hub
description: Learn about the Azure Stack Hub servicing policy and how to keep an integrated system in a supported state.
author: sethmanheim

ms.topic: article
ms.date: 02/07/2020
ms.author: sethm
ms.reviewer: harik
ms.lastreviewed: 01/11/2020

# Intent: As an Azure Stack operator, I want to learn about servicing policy and how to keep an integrated system supported.
# Keyword: servicing policy azure stack

---


# Azure Stack Hub servicing policy

This article describes the servicing policy for Azure Stack Hub integrated systems and what you must do to keep your system in a supported state.

## Download update packages for integrated systems

Microsoft releases both full monthly update packages as well as hotfix packages to address specific issues.

Monthly update packages are hosted in a secure Azure endpoint. You can download them manually using the [Azure Stack Hub Updates downloader tool](https://aka.ms/azurestackupdatedownload). If your scale unit is connected, the update appears automatically in the administrator portal as **Update available**. Full, monthly update packages are well documented at each release. For more information about each release, you can click any release from the [Update package release cadence](#update-package-release-cadence) section of this article.

Hotfix update packages are hosted in the same secure Azure endpoint. You can download them using the embedded links in each of the respective hotfix KB articles; for example, [Azure Stack Hub Hotfix 1.1809.12.114](https://support.microsoft.com/help/4481548/azure-stack-hotfix-1-1809-12-114). Similar to the full, monthly update packages, Azure Stack Hub operators can download the .xml, .bin, and .exe files and import them using the procedure in [Apply updates in Azure Stack Hub](azure-stack-apply-updates.md). Azure Stack Hub operators with connected scale units will see the hotfixes automatically appear in the administrator portal with the message **Update available**.

If your scale unit isn't connected and you want to be notified about each hotfix release, subscribe to the [RSS](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/rss) or [ATOM](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/atom) feed noted in each release.

## Update package types

There are two types of update packages for integrated systems:

- **Microsoft software updates**. Microsoft is responsible for the end-to-end servicing lifecycle for the Microsoft software update packages. These packages can include the latest Windows Server security updates, non-security updates, and Azure Stack Hub feature updates. You can download theses update packages directly from Microsoft.

- **OEM hardware vendor-provided updates**. Azure Stack Hub hardware partners are responsible for the end-to-end servicing lifecycle (including guidance) for the hardware-related firmware and driver update packages. In addition, Azure Stack Hub hardware partners own and maintain guidance for all software and hardware on the hardware lifecycle host. The OEM hardware vendor hosts these update packages on their own download site.

## Update package release cadence

Microsoft expects to release software update packages on a monthly cadence. However, it's possible to have multiple or no update releases in a month. OEM hardware vendors release their updates on an as-needed basis.

Find documentation on how to plan for and manage updates, and how to determine your current version in [Manage updates overview](azure-stack-updates.md).

For information about a specific update, including how to download it, see the release notes for that update:

- [Azure Stack Hub 1910 update](/azure-stack/operator/release-notes?view=azs-1910)
- [Azure Stack Hub 1908 update](/azure-stack/operator/release-notes?view=azs-1908)
- [Azure Stack Hub 1907 update](/azure-stack/operator/release-notes?view=azs-1907)
- [Azure Stack Hub 1906 update](/azure-stack/operator/release-notes?view=azs-1906)

## Hotfixes

Occasionally, Microsoft provides hotfixes for Azure Stack Hub that address a specific issue that's often preventative or time-sensitive. Each hotfix is released with a corresponding Microsoft Knowledge Base article that details the issue, cause, and resolution.

Hotfixes are downloaded and installed just like the regular full update packages for Azure Stack Hub. However, unlike a full update, hotfixes can install in minutes. We recommend Azure Stack Hub operators set maintenance windows when installing hotfixes. Hotfixes update the version of your Azure Stack Hub cloud so you can easily determine if the hotfix has been applied. A separate hotfix is provided for each version of Azure Stack Hub that's still in support. Each fix for a specific iteration is cumulative and includes the previous updates for that same version. You can read more about the applicability of a specific hotfix in the corresponding Knowledge Base article. See the release notes links in the previous section.

For information about currently available hotfixes, see the release notes for that update:

- [Azure Stack Hub 1910 hotfix](/azure-stack/operator/release-notes?view=azs-1910#hotfixes)
- [Azure Stack Hub 1908 hotfix](/azure-stack/operator/release-notes?view=azs-1908#hotfixes-1)
- [Azure Stack Hub 1907 hotfix](/azure-stack/operator/release-notes?view=azs-1907#hotfixes-2)
- [Azure Stack Hub 1906 hotfix](/azure-stack/operator/release-notes?view=azs-1906#hotfixes-3)

## Keep your system under support

For your Azure Stack Hub instance to remain in a supported state, the instance must run the most recently released update version or run either of the two preceding update versions.

Hotfixes aren't considered major update versions. If your Azure Stack Hub instance is behind by *more than two updates*, it's considered out of compliance. You must update to at least the minimum supported version to receive support.

For example, if the most recently available update version is 1904, and the previous two update packages were versions 1903 and 1902, both 1902 and 1903 remain in support. However, 1901 is out of support. The policy holds true when there's no release for a month or two. For example, if the current release is 1807 and there was no 1806 release, the previous two update packages of 1805 and 1804 remain in support.

Microsoft software update packages are non-cumulative and require the previous update package or hotfix as a prerequisite. If you decide to defer one or more updates, consider the overall runtime if you want to get to the latest version.

## Get support

Azure Stack Hub follows the same support process as Azure. Enterprise customers can follow the process described in [How to create an Azure support request](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request). If you're a customer of a Cloud Solution Provider (CSP), contact your CSP for support. For more information, see the [Azure Support FAQs](https://azure.microsoft.com/support/faq/).

For help troubleshooting update issues, see [Best practices for troubleshooting Azure Stack Hub patch and update issues](azure-stack-updates-troubleshoot.md).

## Next steps

- [Manage updates in Azure Stack Hub](azure-stack-updates.md)
- [Best practices for troubleshooting Azure Stack Hub patch and update issues](azure-stack-updates-troubleshoot.md)

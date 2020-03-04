---
title: Troubleshoot updates in Azure Stack Hub 
description: As an Azure Stack Hub operator, learn how to resolve issues with update so that Azure Stack Hub can return to production as quickly as possible. 
author: IngridAtMicrosoft

ms.topic: article
ms.date: 09/23/2019
ms.author: inhenkel
ms.lastreviewed: 09/23/2019
ms.reviewer: ppacent

# Intent: Notdone: As a < type of user >, I want < what? > so that < why? >
# Keyword: Notdone: keyword noun phrase

---

# Best practices for troubleshooting Azure Stack Hub patch and update issues

This article provides an overview of best practices for troubleshooting Azure Stack Hub patch and update issues as well as remediations to common patch and update issues.


The Azure Stack Hub patch and update process is designed to allow operators to apply update packages in a consistent, streamlined way. While uncommon, issues can occur during patch and update process. The following steps are recommended should you encounter an issue during the patch and update process:

0. **Prerequisites**: Be sure that you have followed the [Update Activity Checklist](release-notes-checklist.md) and have [Configured Automatic Log Collection](azure-stack-configure-automatic-diagnostic-log-collection.md).
1. Follow the remediation steps in the failure alert created when your update failed.
2. Review the [Common Azure Stack Hub patch and update issues](#common-azure-stack-hub-patch-and-update-issues) and take the recommended actions if your issue is listed.
3. If you have been unable to resolve your issue with the above steps, create an [Azure Stack Hub support ticket](azure-stack-help-and-support-overview.md). Be sure you have [logs collected](https://docs.microsoft.com/azure-stack/operator/azure-stack-configure-on-demand-diagnostic-log-collection) for the timespan that the issue occurred.

## Common Azure Stack Hub patch and update issues

*Applies to: Azure Stack Hub integrated systems*

### PreparationFailed

**Applicable**: This issue applies to all supported releases.

**Cause**: When attempting to install the Azure Stack Hub update, the status for the update might fail and change state to `PreparationFailed`. For internet-connected systems this is usually indicative of the update package being unable to download properly due to a weak internet connection. 

**Remediation**: You can work around this issue by clicking **Install now** again. If the problem persists, we recommend manually uploading the update package by following the [Install updates](azure-stack-apply-updates.md?#install-updates-and-monitor-progress) section.

**Occurrence**: Common

## Next steps

- [Update Azure Stack Hub](azure-stack-updates.md)  
- [Microsoft Azure Stack Hub help and support](azure-stack-help-and-support-overview.md)

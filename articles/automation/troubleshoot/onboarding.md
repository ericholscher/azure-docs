---
title: Troubleshooting the onboarding of Azure Automation management solutions
description: Learn how to troubleshoot solution onboarding errors.
services: automation
author: mgoedtel
ms.author: magoedte
ms.date: 05/22/2019
ms.topic: conceptual
ms.service: automation
manager: carmonm
---
# Troubleshoot solution onboarding

You might receive errors when onboarding the Update Management solution or the Change Tracking and Inventory solution. This article describes the various errors that might occur and how to resolve them.

## Known issues

### <a name="node-rename"></a>Scenario: Renaming a registered node requires unregister or register again

#### Issue

A node is registered to Azure Automation and then the operating system computer name is changed. Reports from the node
continue to appear with the original name.

#### Cause

Renaming registered nodes does not update the node name in Azure Automation.

#### Resolution

Unregister the node from Azure Automation State Configuration and then register it again. Reports published to the 
service before that time will no longer be available.

### <a name="resigning-cert"></a>Scenario: Re-signing certificates via https proxy is not supported

#### Issue

When connecting through a proxy solution that terminates HTTPS traffic and then re-encrypts the traffic using a new certificate, the service does not allow the connection.

#### Cause

Azure Automation does not support re-signing certificates used to encrypt traffic.

#### Resolution

There is currently no workaround for this issue.

## General errors

### <a name="missing-write-permissions"></a>Scenario: Onboarding fails with the message - The solution cannot be enabled

#### Issue

You receive one of the following messages when you attempt to onboard a VM to a solution:

```error
The solution cannot be enabled due to missing permissions for the virtual machine or deployments
```

```error
The solution cannot be enabled on this VM because the permission to read the workspace is missing
```

#### Cause

This error is caused by incorrect or missing permissions on the VM or workspace, or for the user.

#### Resolution

Ensure that you have correct [permissions needed to onboard machines](../automation-role-based-access-control.md#onboarding-permissions) and then try to onboard the solution again. If you receive the error `The solution cannot be enabled on this VM because the permission to read the workspace is missing`, ensure that you have the `Microsoft.OperationalInsights/workspaces/read` permission to be able to find if the VM is onboarded to a workspace.

### <a name="diagnostic-logging"></a>Scenario: Onboarding fails with the message: Failed to configure automation account for diagnostic logging

#### Issue

You receive the following message when you attempt to onboard a VM to a solution:

```error
Failed to configure automation account for diagnostic logging
```

#### Cause

This error can be caused if the pricing tier doesn't match the subscription's billing model. See [Monitoring usage and estimated costs in Azure Monitor](https://aka.ms/PricingTierWarning).

#### Resolution

Create your Log Analytics workspace manually and repeat the onboarding process to select the workspace created.

### <a name="computer-group-query-format-error"></a>Scenario: ComputerGroupQueryFormatError

#### Issue

This error code means that the saved search computer group query used to target the solution is not formatted correctly. 

#### Cause

You might have altered the query, or the system might have altered it.

#### Resolution

You can delete the query for the solution and then onboard the solution again, which recreates the query. The query can be found in your workspace, under **Saved searches**. The name of the query is **MicrosoftDefaultComputerGroup**, and the category of the query is the name of the associated solution. If multiple solutions are enabled, the **MicrosoftDefaultComputerGroup** query shows multiple times under **Saved Searches**.

### <a name="policy-violation"></a>Scenario: PolicyViolation

#### Issue

This error code indicates that the deployment failed due to violation of one or more policies.

#### Cause 

A policy is blocking the operation from completing.

#### Resolution

In order to successfully deploy the solution, you must consider altering the indicated policy. As there are many different types of policies that can be defined, the changes required depend on the policy that is violated. For example, if a policy is defined on a resource group that denies permission to change the contents of some contained resources, you might choose one of these fixes:

* Remove the policy altogether.
* Try to onboard the solution to a different resource group.
* Re-target the policy to a specific resource, for example, an Automation account.
* Revise the set of resources that the policy is configured to deny.

Check the notifications in the top right corner of the Azure portal or navigate to the resource group that contains your Automation account and select **Deployments** under **Settings** to view the failed deployment. To learn more about Azure Policy, see [Overview of Azure Policy](../../governance/policy/overview.md?toc=%2fazure%2fautomation%2ftoc.json).

### <a name="unlink"></a>Scenario: Errors trying to unlink a workspace

#### Issue

You receive the following error when trying to unlink a workspace:

```error
The link cannot be updated or deleted because it is linked to Update Management and/or ChangeTracking Solutions.
```

#### Cause

This error occurs when you still have solutions active in your Log Analytics workspace that depend on your Automation account and Log Analytics workspace being linked.

### Resolution

Remove the following solutions from your workspace if you are using them:

* Update Management
* Change Tracking and Inventory
* Start/Stop VMs during off-hours

Once you remove the solutions, you can unlink your workspace. It's important to clean up any existing artifacts from these solutions from your workspace and your Automation account 

* For Update Management, remove Update Deployments (Schedules) from your Automation account.
* For Start/Stop VMs during off-hours, remove any locks on solution components in your Automation account under **Settings** > **Locks**. See [Remove the Start/Stop VMs during off-hours solution](../automation-solution-vm-management.md#remove-the-solution).

## <a name="mma-extension-failures"></a>Log Analytics for Windows extension failures

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)] 

An installation of the Log Analytics agent for Windows extension can fail for a variety of reasons. The following section describes onboarding issues that can cause failures during deployment of the Log Analytics agent for Windows extension.

>[!NOTE]
>Log Analytics agent for Windows is the name used currently in Azure Automation for the Microsoft Monitoring Agent (MMA).

### <a name="webclient-exception"></a>Scenario: An exception occurred during a WebClient request

The Log Analytics for Windows extension on the VM is unable to communicate with external resources and the deployment fails.

#### Issue

The following are examples of error messages that are returned:

```error
Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage.
```

```error
'Manifest download error from https://<endpoint>/<endpointId>/Microsoft.EnterpriseCloud.Monitoring_MicrosoftMonitoringAgent_australiaeast_manifest.xml. Error: UnknownError. An exception occurred during a WebClient request.
```

#### Cause

Some potential causes to this error are:

* A proxy configured in the VM only allows specific ports.
* A firewall setting has blocked access to the required ports and addresses.

#### Resolution

Ensure that you have the proper ports and addresses open for communication. For a list of ports and addresses, see [planning your network](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="transient-environment-issue"></a>Scenario: Install failed because of a transient environment issues

The installation of the Log Analytics for Windows extension failed during deployment because of another installation or action blocking the installation

#### Issue

The following are examples of error messages may be returned:

```error
The Microsoft Monitoring Agent failed to install on this machine. Please try to uninstall and reinstall the extension. If the issue persists, please contact support.
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1618'
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.2) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.2\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1601'
```

#### Cause

Some potential causes to this error are:

* Another installation is in progress.
* The system is triggered to reboot during template deployment.

#### Resolution

This error is transient in nature. Retry the deployment to install the extension.

### <a name="installation-timeout"></a>Scenario: Installation timeout

The installation of the Log Analytics for Windows extension didn't complete because of a timeout.

#### Issue

The following is an example of an error message that might be returned:

```error
Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 15614
```

#### Cause

This type of error occurs because the VM is under a heavy load during installation.

### Resolution

Try to install the Log Analytics agent for Windows extension when the VM is under a lower load.

## Next steps

If you don't see your problem above or can't resolve your issue, try one of the following channels for additional support:

* Get answers from Azure experts through [Azure Forums](https://azure.microsoft.com/support/forums/).
* Connect with [@AzureSupport](https://twitter.com/azuresupport), the official Microsoft Azure account for improving customer experience by connecting the Azure community to the right resources: answers, support, and experts.
* File an Azure support incident. Go to the [Azure support site](https://azure.microsoft.com/support/options/) and select **Get Support**.

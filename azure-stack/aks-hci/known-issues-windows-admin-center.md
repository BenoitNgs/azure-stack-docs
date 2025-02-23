---
title: Known issues in Windows Admin Center for Azure Kubernetes Service on Azure Stack HCI 
description: Known issues in Windows Admin Center for Azure Kubernetes Service on Azure Stack HCI 
author: mattbriggs
ms.topic: troubleshooting
ms.date: 09/22/2021
ms.author: mabrigg 
ms.lastreviewed: 1/14/2022
ms.reviewer: abha

---
# Fix issues in Windows Admin Center

This article describes known issues with Windows Admin Center in AKS on Azure Stack HCI. You can also review [upgrade](known-issues.md) and [installation](known-issues-installation.md) issues and errors.

## During deployment the error `No match was found for the specified search criteria for the provider 'NuGet'` appears
This error appears when deploying through Windows Admin Center. The package provider requires the `PackageManagement` and `Provider` tags. You should check if the specified package has tags error when attempting a deployment through Windows Admin Center. 

This is an error occurs from PowerShell and states that there are internet connectivity issues. PowerShell is trying to install the pre-requisites package and is unable to install it. You should check to make sure the server or failover cluster has internet connectivity and then start a fresh installation.

In Stage 2.1: System Validation, there may be an error when you hit install: `No match was found for the specified search criteria for the provider 'NuGet'. The package provider requires 'PackageManagement' and 'Provider' tags. Please check if the specified package has the tags.`
For the PowerShell commands, please complete them on the node(s) that you are trying to deploy to. You will have to manually install the NuGet using the PowerShell command below: 
```powershell-interactive
Install-PackageProvider -Name NuGet -Scope CurrentUser -Force

```
After running that command, please close all open PowerShell windows and try again in Windows Admin Center.

## `Install-Module` was not recognized

In stage 2.1: System Validation, you may get an error: `The item 'Install-Module' was not recognized as the name of a cmdlet, function, script file, or runnable program. Please check the spelling of the name, if the path is included, make sure the path is correct and try again` when you install. For the PowerShell commands, please complete them on the node(s) that you are trying to deploy to. Please run the following command (ensure your PowerShell version is at least 5.1) to resolve:
```powershell-interactive
Install-Module -Name PowershellGet -Repository PSGallery -Force -Confirm:$false -SkipPublisherCheck
```

Then run these commands if you encounter any errors from the first command: 
```powershell-interactive
Get-PSRepository
Register-PSRepository -Default
```

## Unable to find repository 'PSGallery'

In stage 2.1: System Validation, you may encounter this error: `Unable to find repository 'PSGallery'. Use Get-PSRepository to see all available repositories.` when you click install. For the PowerShell commands, please complete them on the node(s) that you are trying to deploy to. Please unregister and reregister the PSRepository in an administrative PowerShell window. Close all PowerShell windows afterward.
```powershell-interactive
Unregister-PSRepository -Name 'PSGallery'
Register-PSRepository -Default
```

Then, please uninstall and reinstall PowerShellGet in an administrative PowerShell window. Close all PowerShell windows afterwards.
```powershell-interactive
Uninstall-Module PowerShellGet
Install-Module PowerShellGet -Force
```

After this, please go back to Windows Admin Center and retry.

## Access is denied

In Stage 2.1: Basic Step Component, you may encounter this error: `Connecting to remote server *** failed with the following error message : Access is denied. For more information, see the about_Remote_Troubleshooting Help topic.` when trying to use your credentials for your server node(s).
Please first make sure that the account/credentials added is an administrative account on the machine. Then verify that PSRemoting is enabled and remote hosts are trusted. You can do this with the following PowerShell commands:
```powershell-interactive
Enable-PSRemoting -Force 
winrm quickconfig 
```

If you are still encountering issues, please follow this [troubleshooting guide](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/about/about_remote_troubleshooting?view=powershell-7.2).

## When updating the Kubernetes version, the update page shows the update is still processing when the update is completed
If you have workload clusters with Kubernetes version 1.19.9 installed, and then use Windows Admin Center to update them to Kubernetes version 1.19.11, the Kubernetes update page continues to show that the update is still in process. However, if you run [Get-AksHciCluster](./reference/ps/get-akshcicluster.md), the output shows that the update is complete, and if you open Windows Admin Center in a new tab, the cluster is updated to 1.19.11 in the **Kubernetes clusters** list. You can ignore this issue as the update process did complete.

## Restarting Azure Stack HCI nodes causes timing issue
Restarting the Azure Stack HCI cluster nodes hosting the management cluster and workload clusters may cause the workload clusters to disappear from the WAC dashboard. To work around this issue, pause and drain the nodes before you plan to restart them. Sometimes, the workload clusters may just take longer to appear in the dashboard.

[ ![Deployment: Connecting to remote server localhost failed.](media/known-issues-windows-admin-center/wac-restart-to-resolve-timing-issues.png) ](media/known-issues-windows-admin-center/wac-restart-to-resolve-timing-issues.png#lightbox)

## Issues occurred when registering the Windows Admin Center gateway with Azure
If you've just created a new Azure account and haven't signed in to the account on your gateway machine, you might experience problems with registering your Windows Admin Center gateway with Azure. To mitigate this problem, sign in to your Azure account in another browser tab or window, and then register the Windows Admin Center gateway to Azure.

## Deployment: Connecting to remote server localhost failed
The AKS host cluster deployment fails at system checks with a WinRM service error. Try applying the solutions suggested in the [Manual troubleshooting](../hci/manage/troubleshoot-credssp.md#manual-troubleshooting). 

![Connecting to remote server localhost failed.](media/known-issues-windows-admin-center/wac-known-issue-description-auto-generated.png)

## Networking field names are inconsistent in the Windows Admin Center portal

There are inconsistencies in network field names showing up in the host cluster deployment flow, and the workload cluster deployment flow.

## The error `Cannot index into a null array` appears when creating an Arc enabled workload cluster
This error appears when moving from PowerShell to Windows Admin Center to create an Arc enabled workload cluster. You can safely ignore this error as it is part of the validation step, and the cluster has already been created. 

## Recovering from a failed AKS on Azure Stack HCI deployment
If you're experiencing deployment issues or want to reset your deployment, make sure you close all Windows Admin Center instances connected to Azure Kubernetes Service on Azure Stack HCI before running [Uninstall-AksHci](./reference/ps/uninstall-akshci.md) from a PowerShell administrative window.

## Cannot deploy AKS to an environment that has separate storage and compute clusters
Windows Admin Center will not deploy Azure Kubernetes Service to an environment with separate storage and compute clusters as it expects the compute and storage resources to be provided by the same cluster. In most cases, it will not find CSVs exposed by the compute cluster and will refuse to continue with deployment.

## Windows Admin Center doesn't have an Arc offboarding experience
Windows Admin Center does not currently have a process to off board a cluster from Azure Arc. To delete Arc agents on a cluster that has been destroyed, navigate to the resource group of the cluster in the Azure portal, and manually delete the Arc content. To delete Arc agents on a cluster that is still up and running, you should run the following command:

```azurecli
az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
``` 
 
> [!NOTE]
> If you use the Azure portal to delete the Azure Arc-enabled Kubernetes resource, it removes any associated configuration resources, but does not remove the agents running on the cluster. Best practice is to delete the Azure Arc-enabled Kubernetes resource using `az connectedk8s delete` instead of Azure portal.

## Cannot connect Windows Admin Center to Azure when creating a new Azure App ID
If you're unable to connect Windows Admin Center to Azure because you can't automatically create and use an Azure App ID on the gateway, create an Azure App ID and assign it the right permissions on the portal. Then, select **Use existing in the gateway**. For more information, visit [connecting your gateway to Azure.](/windows-server/manage/windows-admin-center/azure/azure-integration).

## Only the user who set up the AKS host can create clusters
When deploying Azure Kubernetes Service on Azure Stack HCI through Windows Admin Center, only the user who set up the AKS host can create Kubernetes clusters. To work around this issue, copy the _wssd_ folder from the profile of the user who set up the AKS host to the profile of the user who will be creating the new Kubernetes clusters.

## A timeout error appears when trying to connect an AKS workload cluster to Azure Arc through Windows Admin Center
Sometimes, due to network issues, Windows Admin Center times out an Arc connection. Use the PowerShell command [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md) to connect the AKS workload cluster to Azure Arc while we actively work on improving the user experience.

## Error occurs when attempting to use Windows Admin Center
For CredSSP to function successfully in the Cluster Create wizard, Windows Admin Center must be installed and used by the same account. If you install Windows Admin Center with one account and try to use it with another, you'll get errors.

## On Windows Admin Center, the message `error occurred while creating service principal` appears while installing an AKS host on Azure Stack HCI
You will get this error if you have disabled pop-ups. Google Chrome blocks pop-ups by default, and therefore, the Azure sign-in pop-up is blocked and causes the service principal error.

## The Setup or Cluster Create wizard displays an error about a wrong configuration
If you receive an error in either wizard about a wrong configuration, perform cluster cleanup operations. These operations might involve removing the `C:\Program Files\AksHci\mocctl.exe` file.

## When multiple versions of PowerShell modules are installed, Windows Admin Center does not pick the latest version
If you have multiple versions of the PowerShell modules installed (for example, 0.2.26, 0.2.27, and 0.2.28), Windows Admin Center may not use the latest version (or the one it requires). Make sure you have only one PowerShell module installed. You should uninstall all unused PowerShell versions of the PowerShell modules and leave just one installed. More information on which Windows Admin Center version is compatible with which PowerShell version can be found in the AKS on Azure Stack HCI [release notes.](https://github.com/Azure/aks-hci/releases).

## A WinRM error is displayed when creating a new workload cluster

When switching from DHCP to static IP, Windows Admin Center displayed an error that said the WinRM client cannot process the request. This error also occurred outside of Windows Admin Center. WinRM broke when static IP addresses were used, and the servers were not registering a Service Principal Name (SPN) when moving to static IP addresses. 

To resolve this issue, use the **SetSPN** command to create the SPN. From a command prompt on the Windows Admin Center gateway, run the following command: 

```
Setspn /Q WSMAN/<FQDN on the Azure Stack HCI Server> 
```

Next, if any of the servers in the environment return the message `No Such SPN Found`, then log in to that server and run the following commands:  

```
Setspn /S WSMAN/<server name> <server name> 
Setspn /S WSMAN/<FQDN of server> <server name> 
```

Finally, on the Windows Admin Center gateway, run the following to ensure that it gets new server information from the domain controller:

```
Klist purge 
```

## Troubleshoot CredSSP issues

When deploying AKS on Azure Stack HCI using Windows Admin Center, and the deployment hangs for an extended period, you might be having Credential Security Support Provider (CredSSP) or connectivity problems. Try the following steps to troubleshoot your deployment:
 
1. On the machine running Windows Admin Center, run the following command in a PowerShell window: 

   ```PowerShell
      Enter-PSSession <servername>
   ```
2. If this command succeeds, you can connect to the server and there's no connectivity issue.
    
3. If you're having CredSSP problems, run this command to test the trust between the gateway machine and the target machine: 

   ```PowerShell
      Enter-PSSession –ComputerName <server> –Credential company\administrator –Authentication CredSSP
   ``` 
   You can also run the following command to test the trust in accessing the local gateway: 

   ```PowerShell
      Enter-PSSession -computer localhost -credential (Get-Credential)
   ``` 

For additional CredSSP troubleshooting tips, see [Troubleshoot CredSSP](../hci/manage/troubleshoot-credssp.md).

## Create Windows Admin Center logs
When you report problems with Windows Admin Center, it's a good idea to attach logs to help the development team diagnose your problem. Errors in Windows Admin Center generally come in one of two forms: 
- Events that appear in the event viewer on the machine running Windows Admin Center 
- JavaScript problems that surface in the browser console 

To collect logs for Windows Admin Center, use the `Get-SMEUILogs.ps1` script that's provided in the public preview package. 
 
To use the script, run this command in the folder where your script is stored: 
 
```PowerShell
./Get-SMEUILogs.ps1 -ComputerNames [comp1, comp2, etc.] -Destination [comp3] -HoursAgo [48] -NoCredentialPrompt
```
 
The command has the following parameters:
 
- `-ComputerNames`: A list of machines you want to collect logs from.
- `-Destination`: The machine you want to aggregate the logs to.
- `-HoursAgo`: The start time for collecting logs, expressed in hours before the time you run the script.
- `-NoCredentialPrompt`: A switch to turn off the credentials prompt and use the default credentials in your current environment.
 
If you have difficulties running this script, you can run the following command to view the Help text: 
 
```PowerShell
GetHelp .\Get-SMEUILogs.ps1 -Examples
```

## Running an upgrade results in the error: `Error occurred while fetching platform upgrade information`

When running an upgrade in Windows Admin Center, the following error occurred:

`Error occurred while fetching platform upgrade information. RemoteException: No match was found for the specified search criteria and module name 'AksHci'. Try Get-PSRepository to see all available registered module repositories.`

This error message typically occurs when AKS on Azure Stack HCI is deployed in an environment that has a proxy configured. Currently, Windows Admin Center does not have support to install modules in a proxy environment. To resolve this error, set up AKS on Azure Stack HCI [using the proxy PowerShell command](set-proxy-settings.md).

## Incorrect upgrade notification in Windows Admin Center

If you receive an incorrect upgrade notification message `Successfully installed AksHci PowerShell module version null`, the upgrade operation is successful even if the notification is misleading.

![WAC update dashboard doesn't refresh after successful updates.](media/known-issues-windows-admin-center/wac-known-issue-incorrect-notification.png)

## Windows Admin Center update dashboard doesn't refresh after successful updates

After a success upgrade, the Windows Admin Center update dashboard still shows the previous version. Refresh the browser to fix this issue.

![Networking field names inconsistent in WAC portal.](media/known-issues-windows-admin-center/wac-update-shows-previous-version.png)

## Next steps
- [Troubleshooting overview](troubleshoot-overview.md)
- [Installation issues and errors](known-issues-installation.md)
- [Upgrade known issues](known-issues-upgrade.md)

If you continue to run into problems when you're using Azure Kubernetes Service on Azure Stack HCI, you can file bugs through [GitHub](https://aka.ms/aks-hci-issues).

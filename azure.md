## Azure Cheat Sheet

### Refs
- [How to install and configure Azure PowerShell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)
- [Step-by-Step: Capturing Azure Resource Manager (ARM) VNET Gateway Diagnostic Logs](https://blogs.technet.microsoft.com/keithmayer/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/)
- [Diagnose Azure Virtual Network VPN connectivity issues with PowerShell](https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/)

### PowerShell Getting started
```
Install-Module AzureRM
Install-AzureRM
Install-Module Azure
```

Enable running scripts on the system
```
Set-ExecutionPolicy UnRestricted
```

Import and update the module to install necessary functions
```
Import-Module Azure
Import-Module AzureRM
Update-AzureRM
```

Login resource manager
```
Import-AzureRM
Login-AzureRmAccount
```

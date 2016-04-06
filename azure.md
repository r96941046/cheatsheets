## Azure Cheatsheet

### Azure CLI

- [MS Azure CLI Guide](https://azure.microsoft.com/zh-tw/documentation/articles/virtual-machines-command-line-tools/)

- First switch to the correct npm environment

- Help
```
azure help
azure help config
azure help account list
```

- Set to azure resource manager mode
```
azure config mode arm
```

- Login
```
azure login
```

- Info
```
azure resource list
azure account list
```

- Set subscription to use
```
azure account set <subscription-id>
```

- Connect to storage account
```
azure storage account connectionstring show <storage-name> -g <resource-group-name-for-the-storage>
export AZURE_STORAGE_CONNECTION_STRING="<the-connection-string>"
azure storage blob upload test.txt <container-name> example/test.txt
azure storage blob download <container-name> <blob-name> <target-folder-name>
```

### PowerShell Getting started

- [How to install and configure Azure PowerShell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)

- Open PowerShell ISE with admin
```
Install-Module AzureRM
Install-AzureRM
Install-Module Azure
```

- Enable running scripts on the system
```
Set-ExecutionPolicy UnRestricted
```

- Import and update the module to install necessary functions
```
Import-Module Azure
Import-Module AzureRM
Update-AzureRM
```

- Login resource manager
```
Import-AzureRM
Login-AzureRmAccount
```

### Azure site-to-site VPN specs

- [About VPN devices for Site-to-Site VPN Gateway connections](https://azure.microsoft.com/en-us/documentation/articles/vpn-gateway-about-vpn-devices/)

### Azure site-to-site VPN

- [7 Steps to Building Site-to-Site VPN Connections for V2 VNETs using Azure Resource Manager in the NEW Azure Portal](https://blogs.technet.microsoft.com/keithmayer/2015/12/22/7-steps-to-building-site-to-site-vpn-connections-for-v2-vnets-using-azure-resource-manager-in-the-new-azure-portal/)

### Use PowerShell to dump site-to-site VPN logs to storage account

- [Step-by-Step: Capturing Azure Resource Manager (ARM) VNET Gateway Diagnostic Logs](https://blogs.technet.microsoft.com/keithmayer/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/)
- [Diagnose Azure Virtual Network VPN connectivity issues with PowerShell](https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/)

### VPN Routers

- [Draytek Vigor IPsec VPN configuration](http://maxding.blogspot.tw/2014/07/draytek-vigor-2920n-ipsec-vpn.html)
- [Draytek syslog access](https://www.draytek.com/index.php?option=com_k2&view=item&id=2062&Itemid=293&lang=en)

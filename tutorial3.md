# Deploy Applications to a Windows virtual machine in Azure with the Custom Script Extension 

### Create virtual machine
* Set the administrator username and password for the VM.
```
$cred = Get-Credential
```
Create the VM with *New-AzVM* command.
```
New-AzVm `
    -ResourceGroupName "tutorial3ResourceGroupAutomate" `
    -Name "tutorial3VM" `
    -Location "westeurope" `
    -VirtualNetworkName "tutorial3Vnet" `
    -SubnetName "tutorial3Subnet" `
    -SecurityGroupName "tutorial3NetworkSecurityGroup" `
    -PublicIpAddressName "tutorial3PublicIpAddress" `
    -OpenPorts 80 `
    -Credential $cred
```
```
ResourceGroupName        : tutorial3ResourceGroupAutomate
Id                       : /subscriptions/xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/tutorial3ResourceGroupAutomate/provider
s/Microsoft.Compute/virtualMachines/tutorial3VM
VmId                     : aef9505a-4c72-416e-a219-be64181fc2db
Name                     : tutorial3VM
Type                     : Microsoft.Compute/virtualMachines
Location                 : westeurope
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets, AllowExtensionOperations}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorial3vm-ea6767.westeurope.cloudapp.azure.com
```
### Automate IIS install
* Use *Set-AzVMExtension* to install Custom Script Extension. The extension runs *powershell Add-WindowsFeature Web-Server* to install the IIS webserver and then updates the Default.htm page to show the hostname of the VM.

```
Set-AzVMExtension -ResourceGroupName "tutorial3ResourceGroupAutomate" `
    -ExtensionName "IIS" `
    -VMName "tutorial3VM" `
    -Location "westeurope" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
```
```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```
### Test web site
* Obtain the public IP address of the load balancer with *Get-AzPublicIPAddress*
```
Get-AzPublicIPAddress `
    -ResourceGroupName "tutorial3ResourceGroupAutomate" `
    -Name "tutorial3PublicIPAddress" | select IpAddress
```
```
IpAddress
---------
137.117.138.61
```
* You can then enter the public IP address in to a web browser. The website is displayed, including the hostname of the VM.

![alt text]( .\tutorial3_iis.png "Automated IIS install")

### Stop The virtual machine
```
Stop-AzVM `
   -ResourceGroupName "tutorial3ResourceGroupAutomate" `
   -Name "tutorial3VM" -Force
```
```
OperationId : 7c79232c-75d4-482b-befe-c22ccb9b2dcd
Status      : Succeeded
StartTime   : 7/28/2019 1:30:59 AM
EndTime     : 7/28/2019 1:31:47 AM
Error
```
### Delete resource group
```
Remove-AzResourceGroup `
   -Name "tutorial3ResourceGroupAutomate" `
   -Force
```
```
True
```
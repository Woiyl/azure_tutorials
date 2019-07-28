# Create a custom image of an Azure VM with Azure PowerShell
### Prepare VM
To create an image of a virtual machine, you need to prepare the source VM by generalizing it, deallocating it and then making it as generalized with Azure.

### Generalize the Windows VM using Sysprep 
1. Connect to the virtual machine
2. Change the directory to *%windir%\system32\sysprep*, and then run *sysprep.exe*.
3. In the __System Preparation Tool__ dialog box, select Enter __System Out-of-Box Experience (OOBE)__, and make sure that the __Generalize__ check box is selected.
4. In __Shutdown Options__, select __Shutdown__ and then click __OK__.
5. When Sysprep completes, it shuts down the virtual machine. __Do not restart the VM__.

### Create a Virtual machine:
This time use Azure location in North Europe region (Ireland)
```
$cred = Get-Credential

New-AzVm `
    -ResourceGroupName "tutorial4ResourceGroupVM" `
    -Name "tutorial4VM" `
    -Location "northeurope" `
    -VirtualNetworkName "tutorial4Vnet" `
    -SubnetName "tutorial4Subnet" `
    -SecurityGroupName "tutorial4NetworkSecurityGroup" `
    -PublicIpAddressName "tutorial4PublicIpAddress" `
    -Credential $cred

```
```
ResourceGroupName        : tutorial4ResourceGroupVM
Id                       : /subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/tutorial4ResourceGroupVM/providers/Micr
osoft.Compute/virtualMachines/tutorial4VM
VmId                     : 5bd9f929-f3ba-4777-84f3-d7ec2a8cdbc4
Name                     : tutorial4VM
Type                     : Microsoft.Compute/virtualMachines
Location                 : northeurope
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets, AllowExtensionOperations}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorial4vm-b83b32.northeurope.cloudapp.azure.com
```
### Deallocate and mark the VM as generalized
Deallocate the VM using *Stop-AzVM*
```
Stop-AzVM `
   -ResourceGroupName tutorial4ResourceGroupVM `
   -Name tutorial4VM -Force
```
```
OperationId : 6cd5e7a9-fe33-487d-842b-9b3587de7f95
Status      : Succeeded
StartTime   : 7/28/2019 7:44:28 AM
EndTime     : 7/28/2019 7:53:05 AM
Error       :
```
Set the status of the virtual machine to *-Generalized* using *Set-AzVm*
```
Set-AzVM `
   -ResourceGroupName tutorial4ResourceGroupVM `
   -Name tutorial4VM -Generalized
```
```
OperationId :
Status      : Succeeded
StartTime   : 7/28/2019 8:11:48 AM
EndTime     : 7/28/2019 8:11:49 AM
Error       :
```
### Create the image
Create an image of the VM by using *New-AzImageConfig* and *New-AzImage*
```
$vm = Get-AzVM `
   -Name tutorial4VM `
   -ResourceGroupName tutorial4ResourceGroupVM
```
* Create the image configuration
```
$image = New-AzImageConfig `
   -Location northeurope `
   -SourceVirtualMachineId $vm.ID
```
* Create the image
```
New-AzImage `
   -Image $image `
   -ImageName tutorial4Image `
   -ResourceGroupName tutorial4ResourceGroupVM
```
```

ResourceGroupName    : tutorial4ResourceGroupVM
SourceVirtualMachine : Microsoft.Azure.Management.Compute.Models.SubResource
StorageProfile       : Microsoft.Azure.Management.Compute.Models.ImageStorageProfile
ProvisioningState    : Succeeded
HyperVGeneration     : V1
Id                   : /subscriptions/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/tutorial4ResourceGroupVM/providers/Microsof
                       t.Compute/images/tutorial4Image
Name                 : tutorial4Image
Type                 : Microsoft.Compute/images
Location             : northeurope
Tags                 : {}
```
### Create VMs from the image
When you use a Marketplace image, you have to provide the information about the image, image provider, offer, SKU, and version. Using the simplified parameter set for the New-AzVM cmdlet, you just need to provide the name of the custom image as long as it is in the same resource group.
```
New-AzVm `
    -ResourceGroupName "tutorial4ResourceGroupVM" `
    -Name "tuto4VM" `
    -ImageName "tutorial4Image" `
    -Location "northeurope" `
    -VirtualNetworkName "tuto4Vnet" `
    -SubnetName "tuto4Subnet" `
    -SecurityGroupName "tuto4NetworkSecurityGroup" `
    -PublicIpAddressName "tuto4ImagePublicIpAddress" `
    -OpenPorts 3389
```
```
```

### Image managment

* List all images by name
```
$images = Get-AzResource -ResourceType Microsoft.Compute/images 
$images.name
```
```
tutorial4Image
```
* Deletes an image from the resource group.
```
Remove-AzImage `
    -ImageName tutorial4Image `
	-ResourceGroupName tutorial4ResourceGroupVM
```

### Delete resource group
```
Remove-AzResourceGroup `
   -Name "tutorial4ResourceGroupVM" `
   -Force
```
```
True
```

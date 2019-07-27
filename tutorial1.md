# Create and Manage Windows VMs with Azure PowerShell

### Create a resource group:
```
New-AzResourceGroup `
   -ResourceGroupName "tutorial1ResourceGroupVM" `
   -Location "westeurope"
```   

### Create a Virtual machine:
```
$cred = Get-Credential

New-AzVm `
    -ResourceGroupName "tutorial1ResourceGroupVM" `
    -Name "tutorial1VM" `
    -Location "westeurope" `
    -VirtualNetworkName "tutorial1Vnet" `
    -SubnetName "tutorial1Subnet" `
    -SecurityGroupName "tutorial1NetworkSecurityGroup" `
    -PublicIpAddressName "tutorial1PublicIpAddress" `
    -Credential $cred

```
```
ResourceGroupName        : tutorial1ResourceGroupVM
Id                       : /subscriptions/xxxxxx-xxxx-xxxxxxx-xxxxxx-xxxxxxxxx/resourceGroups/tutorial1ResourceGroupVM/providers/Micr
osoft.Compute/virtualMachines/tutorial1VM
VmId                     : 5b83c0cd-46ac-4c2f-9ab4-eae293c448e0
Name                     : tutorial1VM
Type                     : Microsoft.Compute/virtualMachines
Location                 : westeurope
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets, AllowExtensionOperations}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorial1vm-9ec2dd.westeurope.cloudapp.azure.com
```

### Connect to your VM
```
Get-AzPublicIpAddress `
   -ResourceGroupName "tutorial1ResourceGroupVM"  | Select IpAddress
```
```
IpAddress
40.118.66.30
```
Use Remote desktop connection from your local machine by specifying the ip address you have received. Then you are connected to your VM.


### Understand marketplace images
List all the available images in West Europe location

```
 Get-AzVMImageOffer `
    -Location "westeurope" `
    -PublisherName "MicrosoftWindowsServer"
```
```
Offer                                     PublisherName          Location   Id
-----                                     -------------          --------   --
19h1gen2servertest                        MicrosoftWindowsServer westeurope /Subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Pr...
server2016gen2testing                     MicrosoftWindowsServer westeurope /Subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Pr...
servertesting                             MicrosoftWindowsServer westeurope /Subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Pr...
```
* Filter on the publisher and the offer:

```
Get-AzVMImageSku `
   -Location "westeurope" `
   -PublisherName "MicrosoftWindowsServer" `
   -Offer "WindowsServer"
```
```
Skus    Offer   PublisherName  Location   Id
                                           
2008-R2-SP1 WindowsServer   MicrosoftWindowsServer  westeurope /Subscriptions/xxxx-xxx-xxx-...
```

### Create a second vm by specifying the image name
```
New-AzVm `
    -ResourceGroupName "tutorial1ResourceGroupVM" `
    -Name "tutorial1VM2" `
    -Location "westeurope" `
    -VirtualNetworkName "tutorial1Vnet" `
    -SubnetName "tutorial1Subnet" `
    -SecurityGroupName "tutorial1NetworkSecurityGroup" `
    -PublicIpAddressName "tutorial1PublicIpAddress2" `
    -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
    -Credential $cred `
    -AsJob
```
```
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running... AzureLongRun... Running       True            localhost            New-AzVM
```
* Check that the second VM is provisioned
```
Get-AzPublicIpAddress `
-ResourceGroupName "tutorial1ResourceGroupVM"  | Select IpAddress

IpAddress
---------
40.118.66.30
104.45.17.139
```

### Find available VM sizes

```
Get-AzVMSize -Location "Westeurope"
```
```
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_A0                        1        768                1        1047552                20480
Standard_A1                        1       1792                2        1047552                71680
Standard_A2                        2       3584                4        1047552               138240
```

### Resize a VM

* Check if the size that we want to resize to is available. 

```
Get-AzVMSize -ResourceGroupName "tutorial1ResourceGroupVM" -VMName "tutorial1VM"
```
```
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_B1ls                      1        512                2        1047552                 1024
Standard_B1ms                      1       2048                2        1047552                 4096
Standard_B1s                       1       1024                2        1047552                 2048
Standard_B2ms                      2       8192                4        1047552                16384
Standard_B2s                       2       4096                4        1047552                 8192
Standard_B4ms                      4      16384                8        1047552                32768
```

* Issue the VM resize command
```
$vm = Get-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM"  `
   -VMName "tutorial1VM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "tutorial1ResourceGroupVM"
```
```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK


```
* Deallocating before resizing
```
Stop-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM" `
   -Name "tutorial1VM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM"  `
   -VMName "tutorial1VM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "tutorial1ResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM"  `
   -Name $vm.name
```
```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK


OperationId : bc6cbcd9-ac06-4dc7-9420-50eba8d6ec4b
Status      : Succeeded
StartTime   : 7/27/2019 10:33:09 PM
EndTime     : 7/27/2019 10:33:28 PM
Error       :
```

### VM power states

```
Get-AzVM `
    -ResourceGroupName "tutorial1ResourceGroupVM" `
    -Name "tutorial1VM" `
    -Status | Select @{n="Status"; e={$_.Statuses[1].Code}}
```
```
Status
------
PowerState/running
```

### Stop a VM
```
Stop-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM" `
   -Name "tutorial1VM" -Force
```

```
OperationId : a061f315-f6a7-4a62-a863-b31c0dfbbe77
Status      : Succeeded
StartTime   : 7/27/2019 10:45:47 PM
EndTime     : 7/27/2019 10:46:34 PM
Error       :
```
### Start a VM
```
Start-AzVM `
   -ResourceGroupName "tutorial1ResourceGroupVM" `
   -Name "tutorial1VM"
```
```
OperationId : dcff22ba-c279-4fe1-ace2-130b18df1051
Status      : Succeeded
StartTime   : 7/27/2019 10:51:10 PM
EndTime     : 7/27/2019 10:51:38 PM
Error       :
```
### Delete resource group
```
Remove-AzResourceGroup `
   -Name "tutorial1ResourceGroupVM" `
   -Force
```
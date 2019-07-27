# Manage Azure disks with Azure PowerShell
### Create and attach disks
* Create the virtual machine with *New-AzVM*. You will be prompted to enter a username and password for the administrators account of the VM.

```
New-AzVm `
    -ResourceGroupName "tutorial2ResourceGroupDisk" `
    -Name "tutorial2VM" `
    -Location "westeurope" `
    -VirtualNetworkName "tutorial2Vnet" `
    -SubnetName "tutorial2Subnet" `
    -SecurityGroupName "tutorial2NetworkSecurityGroup" `
    -PublicIpAddressName "tutorial2PublicIpAddress"
```
```
cmdlet New-AzVM at command pipeline position 1
Supply values for the following parameters:
Credential


ResourceGroupName        : tutorial2ResourceGroupDisk
Id                       : /subscriptions/95a127ab-87ec-4762-b1b7-6f2baa46c162/resourceGroups/tutorial2ResourceGroupDisk/providers/Mi
crosoft.Compute/virtualMachines/tutorial2VM
VmId                     : 60be7d1c-2299-4119-93b4-1d285eb6d994
Name                     : tutorial2VM
Type                     : Microsoft.Compute/virtualMachines
Location                 : westeurope
Tags                     : {}
HardwareProfile          : {VmSize}
NetworkProfile           : {NetworkInterfaces}
OSProfile                : {ComputerName, AdminUsername, WindowsConfiguration, Secrets, AllowExtensionOperations}
ProvisioningState        : Succeeded
StorageProfile           : {ImageReference, OsDisk, DataDisks}
FullyQualifiedDomainName : tutorial2vm-afa19c.westeurope.cloudapp.azure.com
```

* Configure a disk that is 128 GB in size using the command *NewAzDiskConfig*
```
$diskConfig = New-AzDiskConfig `
    -Location "westeurope" `
    -CreateOption Empty `
    -DiskSizeGB 128
```

* Create a data disk with the New-AzDisk command.
```
$dataDisk = New-AzDisk `
    -ResourceGroupName "tutorial2ResourceGroupDisk" `
    -DiskName "tutorial2DataDisk" `
    -Disk $diskConfig
```
* Get the virtual machine that you want to add the disk to with the *Get-AzVM* command.
```
$vm = Get-AzVM -ResourceGroupName "tutorial2ResourceGroupDisk" -Name "tutorial2VM"
```
* Add the data disk to the virtual machine configuration using *Add-VMDataDisk* command.
```
$vm = Add-AzVMDataDisk `
    -VM $vm `
    -Name "tutorial2DataDisk" `
    -CreateOption Attach `
    -ManagedDiskId $dataDisk.Id `
    -Lun 1
```
* Update the virtual machine with the *Update-AzVM* command.
```
Update-AzVM -ResourceGroupName "tutorial2ResourceGroupDisk" -VM $vm
```
```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```
### Prepare data disks
* Once a disk has been attached to the virtual machine, We need to manually configure the disk so that it can be used by the operating system.

#### Manual configuration
* Create an RDP connection with the virtual machine using the VM IP address. 
```
Get-AzPublicIpAddress `
-ResourceGroupName "tutorial2ResourceGroupDisk"  | Select IpAddress
```
```
IpAddress
---------
104.40.250.141
```
* Open up PowerShell and run this script in elevated mode.
```
Get-Disk | Where partitionstyle -eq 'raw' |
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -AssignDriveLetter -UseMaximumSize |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "tutorial2DataDisk" -Confirm:$false
```
```
DriveLetter FileSystemLabel   FileSystem DriveType HealthStatus OperationalStatus SizeRemaining   Size
----------- ---------------   ---------- --------- ------------ ----------------- -------------   ----
F           tutorial2DataDisk NTFS       Fixed     Healthy      OK                    127.89 GB 128 GB

```
### Verify data disk
View the StorageProfile for the attached DataDisks. 
```
$vm.StorageProfile.DataDisks
```
```
Name            : tutorial2DataDisk
DiskSizeGB      :
Lun             : 1
Caching         : None
CreateOption    : Attach
SourceImage     :
VirtualHardDisk :
```
```
$vm.StorageProfile.OsDisk
```
```
OsType                  : Windows
EncryptionSettings      :
Name                    : tutorial2VM_OsDisk_1_0e49ca9a23f94f429c5ce034837f7415
Vhd                     :
Image                   :
Caching                 : ReadWrite
WriteAcceleratorEnabled :
DiffDiskSettings        :
CreateOption            : FromImage
DiskSizeGB              : 127
ManagedDisk             : Microsoft.Azure.Management.Compute.Models.ManagedDiskParameters
```
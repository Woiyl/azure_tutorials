# Install Azure PowerShell

## First check the version available on your machine:
```
The version , I have on my machine:
PS C:\Windows\system32> $PSVersionTable.PSVersion
```
```
Major  Minor  Build  Revision
-----  -----  -----  --------
5      1      17763  592
```
### Install Azure PowerShell module
* The command to run on your elavated PowerShell session:
```
PS C:\Windows\system32> *Install-Module -Name Az -AllowClobber*
```
```
NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
'C:\Users\woiyl\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running 'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and import the NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```
```
Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS C:\Windows\system32>
```
* Issue the following command to connect to Azure Cloud
```
PS C:\Windows\system32> Connect-AzAccount
```
```
Account               SubscriptionName TenantId                             Environment
-------               ---------------- --------                             -----------
XYZ@mail.com  XXX-xxx-XXX    xxxx-xxxx-xxx-xxx-xxxxxxxxxxx AzureCloud
```
---
layout: blog
title: PowerShell安装MSOnline模块错误
date: 2021-11-17 08:00:00
tags:
    - PowerShell
    - Azure
    - M365
---


在`Windows Server 2016`上安装`MSOnline`模块时提示错误，具体错误如下：

```
PS C:\> Install-Module MSOnline
需要使用 NuGet 提供程序来继续操作
PowerShellGet 需要使用 NuGet 提供程序“2.8.5.201”或更高版本来与基于 NuGet 的存储库交互。必须在“C:\Program
Files\PackageManagement\ProviderAssemblies”或“C:\Users\administrator.WAFERSYSTEMS1\AppData\Local\PackageManagement\ProviderAssemblies”中提供 NuGet 提供程序。也可以通过运行
'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force' 安装 NuGet 提供程序。是否要让 PowerShellGet 立即安装并导入 NuGet 提供程序?
[Y] 是(Y)  [N] 否(N)  [S] 暂停(S)  [?] 帮助 (默认值为“Y”): y
警告: 无法从 URI“https://go.microsoft.com/fwlink/?LinkID=627338&clcid=0x409”下载到“”。
警告: 无法下载可用提供程序列表。请检查 Internet 连接。
PackageManagement\Install-PackageProvider : 找不到提供程序“NuGet”的指定搜索条件的匹配项。程序包提供程序要求 "PackageManagement" 和 "Provider" 标记。请检查指定的程序包是否具有标记。
所在位置 C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:7405 字符: 21
+ ...     $null = PackageManagement\Install-PackageProvider -Name $script:N ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (Microsoft.Power...PackageProvider:InstallPackageProvider) [Install-PackageProvider]，Exception
    + FullyQualifiedErrorId : NoMatchFoundForProvider,Microsoft.PowerShell.PackageManagement.Cmdlets.InstallPackageProvider

PackageManagement\Import-PackageProvider : 未找到与指定搜索条件和提供程序名称“NuGet”匹配的项目。请尝试运行 'Get-PackageProvider -ListAvailable' 以查看系统中是否存在该提供程序。
所在位置 C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:7411 字符: 21
+ ...     $null = PackageManagement\Import-PackageProvider -Name $script:Nu ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidData: (NuGet:String) [Import-PackageProvider]，Exception
    + FullyQualifiedErrorId : NoMatchFoundForCriteria,Microsoft.PowerShell.PackageManagement.Cmdlets.ImportPackageProvider

警告: 无法从 URI“https://go.microsoft.com/fwlink/?LinkID=627338&clcid=0x409”下载到“”。
警告: 无法下载可用提供程序列表。请检查 Internet 连接。
PackageManagement\Get-PackageProvider : 找不到程序包提供程序“NuGet”。可能尚未导入该提供程序。请尝试使用 'Get-PackageProvider -ListAvailable'。
所在位置 C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:7415 字符: 30
+ ... tProvider = PackageManagement\Get-PackageProvider -Name $script:NuGet ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Microsoft.Power...PackageProvider:GetPackageProvider) [Get-PackageProvider], Exception
    + FullyQualifiedErrorId : UnknownProviderFromActivatedList,Microsoft.PowerShell.PackageManagement.Cmdlets.GetPackageProvider

Install-Module : 需要使用 NuGet 提供程序来与基于 NuGet 的存储库交互。请确保已安装 NuGet 提供程序“2.8.5.201”或更高版本。
所在位置 行:1 字符: 1
+ Install-Module msonline
+ ~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Install-Module]，InvalidOperationException
    + FullyQualifiedErrorId : CouldNotInstallNuGetProvider,Install-Module

```


从提示来看，是由于`NuGet`安装时错误，经过查询，发现使用由于`PowerShell`默认没有使用`TLS1.2`导致，强制指定一下，然后很执行就可以了。指定方法：

```
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;
```

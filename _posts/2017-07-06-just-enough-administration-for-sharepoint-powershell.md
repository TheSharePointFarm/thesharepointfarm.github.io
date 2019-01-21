---
layout: post
title: Just Enough Administration for SharePoint PowerShell
tags: [sp2016]
---

[Just Enough Administration](https://msdn.microsoft.com/en-us/powershell/jea/overview) is a new feature of PowerShell in Windows Server 2016 and WMF 5.x which we can leverage across many enterprise applications, including SharePoint. It allows you to assign rights with "just enough" permission to perform a specific task or function. For example, we might want to allow a group of individuals to execute a timer job, but not change Managed Metadata Service Application settings, drop Content Databases, and so forth. Enter JEA! We'll walk through a simple scenario of how to configure JEA for SharePoint. This scenario can be greatly expanded upon and does not cover all possible scenarios you may want to accomplish via JEA.

The first step to configuring JEA is to create an Active Directory Security Group and add the members to the group for which JEA should apply. The next step is to enable PowerShell Remoting. On your SharePoint server, run `Enable-PSRemoting -Force`. As we'll be using a local account for JEA, explained in more detail below, we'll need to add the machine account as a Shell Administrator.

```powershell
Add-SPShellAdmin 'LAB\SPServer01
```

On your SharePoint server, create a new directory under `C:\Program Files\WindowsPowerShell\Modules\ called Microsoft.SharePoint.PowerShell`. Within that directory, create a new file, `Microsoft.SharePoint.Powershell.psm1` with the below contents.

```powershell
Add-PSSnapin -Name Microsoft.SharePoint.PowerShell -EA 0
```

Next, again in the directory `C:\Program Files\WindowsPowerShell\Modules\`, create a new folder named `SharePointJEA` (the name can be anything, but follow the example on your first run through). Within the `SharePointJEA` directory, create a new directory named `RoleCapabilities`.

We'll create another module. The module file name must match the directory name, or `SharePointJEA.psm1` in our example. Within the SharePointJEA directory, create a `SharePointJEA.psm1` file. In PowerShell, create a module manifest based off of our new module.

```powershell
New-ModuleManifest -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\SharePointJEA.psd1') -RootModule 'SharePointJEA.psm1'
```

We do not need to make any changes to either of these files. The next step is to generate our Role Capabilities file. This file will define what cmdlets the defined group will have access to.

```powershell
New-PSRoleCapabilitiesFile -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\RoleCapabilities\SPJEA.psrc'
```

Open this file in your favorite text editor. You'll see there are quite a few lines commented out. We can either replace those lines with our specified text, or we can just create new line entries. Either route you go, make sure you only have one Key = Value pair for each named key. First, we'll add the ModuleToImport.

```powershell
ModulesToImport = 'Microsoft.SharePoint.PowerShell'
```

This will import our module that loads the SharePoint snapin we previously created in the Modules folder. The next step is to define the cmdlets we want our users to have access to, changing the VisibleCmdlets value. In this example, we'll allow the users to run Get-SPFarm and Get-SPWebApplication.

```powershell
VisibleCmdlets = 'Get-SPFarm', 'Get-SPWebApplication'
```

Save the file, these are the only changes we'll make. The next step is to create our PowerShell Session Configuration file. First, define the roles. This value is based off of the Active Directory Security Group, for example, LAB\SharePointJEA. The RoleCapabilities value, SPJEA, is the same name as our previously created Role Capabilities file. You can have multiple Role Capabilities files with different definitions (e.g. GroupA might have access to Get-SP* cmdlets while GroupB might have access to New-SP* cmdlets). A single group can also have access to multiple Role Capabilities. In this example, we're only specifying a single group and capability file. If you need to specify multiple groups or Role Capabilities, just separate them with a comma.

```powershell
$roles = @{'LAB\SharePointJEA' = @{RoleCapabilities = 'SPJEA'}}
```

Next, create the configuration file. You'll see a few different parameters in this cmdlet. The RunAsVirtualAccount is a parameter which, in essence, creates a Local Administrator session. We need Local Administrator for most, but not all, SharePoint cmdlets. This is the best way to use JEA, although there are other options such as gMSA, which I will not go into here. The TranscriptDirectory parameter is a directory (which must exist) that stores session commands. This allows you to review all executed cmdlets within the JEA sessions. The transcripts are written by LocalSystem and this folder should be locked down to prevent unauthorized access. Lastly, we're passing in our $roles variable to RoleDefinitions.

```powershell
New-PSSessionConfigurationFile -SessionType RestrictedRemoteServer -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\SPJEA.pssc' -RunAsVirtualAccount -TranscriptDirectory 'C:\JEATranscripts\' -RoleDefinitions $roles
```

You can further edit this file via your favorite text editor, if required. Next, make sure you test your session configuration file! This can be accomplished via the below command. It must return True. If it returns False, you have an invalid configuration file.

```powershell
Test-PSSessionConfigurationFile -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\SPJEA.pssc'
```

The last step server-side is to register the configuration file and restart WinRM.

```powershell
Register-PSSessionConfiguration -Name SPJEA -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\SPJEA.pssc'
Restart-Service WinRM
```

On your client workstation, run the following command to create a new remote session scoped to the JEA configuration.

```powershell
Enter-PSSession -ComputerName SPServer01 -ConfigurationName SPJEA
```

When connected, run `Get-Command`. This should only list the default commands that are provided via JEA along with `Get-SPFarm` and `Get-SPWebApplication` from our example. Executing those commands should yield the expected output as if you were running them through a standard PowerShell session.

JEA does have limitations. JEA sessions run in PowerShell [No Language mode](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.core/about/about_language_modes). This means you can't use variables. Instead, you can craft functions that users of the JEA session can call. For example, let's say you wanted to run the simple command `$sa = Get-SPServiceApplication | ?{$_.TypeName -match 'Profile'}; $sa.ApplicationPool`. This would error out in the JEA session. The alternative is to create a function, such as the below function.

```powershell
function Get-SPAppPoolInfo($partialType)
{ 
    $sa = Get-SPServiceApplication | ?{$_.TypeName -match $partialType} 
    $sa.Name 
    $sa.ApplicationPool
}
```

While not best practice, for our example, add this function to the module we previously created, `Microsoft.SharePoint.PowerShell.psm1` at any point below the `Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0` line.

Edit the file `C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\RoleCapabilities\SPJEA.psrc`. Add a new line to expose our function.

```powershell
VisibleFunctions = 'Get-SPAppPoolInfo'
```

Enter a new JEA session and you can run the new function successfully, e.g. `Get-SPAppPoolInfo 'Profile'` or `Get-SPAppPoolInfo 'Metadata'`. This will show us the Application Pool information for the User Profile Service Application and Managed Metadata Service Application.

While most changes to JEA configuration only require one to end and create a new remote session, changes to the Session Configuration File require that you unregister and register the configuration file again. A restart of WinRM may also be required. To complete this, run the below script based off of our example.

```powershell
Unregister-PSSessionConfiguration -Name SPJEA
Register-PSSessionConfiguration -Name SPJEA -Path 'C:\Program Files\WindowsPowerShell\Modules\SharePointJEA\SPJEA.pssc'
Restart-Service WinRM
```

Hopefully this will be useful to segregate personnel who need to have full administrative access to the farm, where JEA provides no value, versus those who need selective access to administer the farm, where JEA can play a valuable role in providing exactly the amount of access they require.
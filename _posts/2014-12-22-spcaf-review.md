---
layout: post
title: SPCAF Review
tags: [news,sp2010,sp2013]
---

Disclaimer: SPCAF is free for Microsoft MVPs and I was provided an MVP license to use this product.


I will be taking a look at the SPCAF Client, Visual Studio integration, and PowerShell for this review. The primary purpose behind SPCAF is for code analysis, correctness, and of course a migration assessment from SharePoint Solutions to SharePoint Apps. SPCAF does not require SharePoint binaries, therefor it can be run on virtually any Windows desktop system, which is great news for IT Professionals and developers creating SharePoint Apps.


I'm going to take a look at my solution, [FoundationSync](https://github.com/Nauplius/FoundationSync). This is a solution geared towards SharePoint Foundation and updates the User Information List for each Site Collection on a daily basis, with the latest version for SharePoint Foundation 2013 pulling pictures from Active Directory or Exchange 2013.

To get started with the SPCAF Client application, on the Home Page, simply drop one or more solutions (.wsp and .app are supported) into the wheel and click the Play button. When solution analysis has completed, it will present a screen similar to below, notating code quality issues, inventory of the solution, internal and external dependencies that your solution takes, and metrics. Notice the Setting next to the project name. This is running with the default rule set, which is an extended rule set. Many issues may be flagged by this rule set that are not applicable to your solution, or that the solution must perform.


![SPCAF_1](/assets/images/2014/12/SPCAF_1.png)


The Code Quality tab presents a very easy to read and filter view. Clicking on any one of the levels, such as Errors, Critical Warnings, and so forth will filter the results to that level.


![SPCAF_2](/assets/images/2014/12/SPCAF_2.png)


Below this screen are the actual listed errors, with a description, resolution, links if applicable. It also displays the portion of code where the rule was violated. In this case, because it is the SPCAF client, the solution is decompiled and may not exactly match the code that was created, but it is close enough for these types of rule violations. In this particular solution, the SPFarm property bag was leveraged to store certain values. In order to create or update a property bag item, SPFarm.Update() must be called. According to SPCAF, this is a rule violation. However, because this was the preferred, and frankly easiest method to store this information, this particular violation isn't applicable. For this rule, SPCAF does provide a help link to understand why this is considered a rule violation. Again, because this violation will be ignored, I can consider decorating the method and suppressing the rule. SPCAF outlines this in the [help documentation](http://docs.spcaf.com/v5/SPCAF_OVERVIEW_800_HOWTOSUPPRESSVIOLATIONS.html) online.

![SPCAF_3](/assets/images/2014/12/SPCAF_3.png)

This is an example of a rule that should be considered. The guidance I've always heard is "dispose of SPSite and SPWeb". Apparently disposing of SPSite.RootWeb should not be done. Again, because this is decompiled, it was not exact, as I was performing a "using(SPWeb web = site.RootWeb){}" in this particular case, but as a developer, it is easy to translate.

![SPCAF_4](/assets/images/2014/12/SPCAF_4.png)

Another example of a warning would be a missing image for a Feature. While this isn't particularly helpful, or worrisome, it is an example of something that can be easily missed and quickly corrected. To conclude the IT Pro or QA toolset, SPCAF recently released a PowerShell module to analyse solutions via PowerShell which can be integrated into an automated process. While the module is free, most of the features require an individual running the cmdlet to have an SPCAF license, although the output of the reports can be shared with anyone.


The PowerShell module can be downloaded from the TechNet Gallery. The package includes two example PowerShell scripts, the module, and a license file (.lic) that can be used to fill in the license information. To see the output of this cmdlet, I've made the reports available here [FoundationSync_SPCAF_Results](/assets/other/2014/12/FoundationSync_SPCAF_Results.zip). The output from PowerShell is included below.

```powershell
PS C:\users\trevor\desktop\spcaf # Import-Module .\SPCAF.PowerShell.dll
PS C:\users\trevor\desktop\spcaf # Invoke-SPCAFCodeAnalysis -InputFiles C:\Users\trevor\desktop\Nauplius.SharePoint.FoundationSync.wsp -Reports DOCX -OutputFile C:\Users\trevor\desktop\FoundationSync.docx -Verbose:$true
VERBOSE: Copying license file from 'C:\users\trevor\desktop\spcaf\YourLicenseForFullFeatures.lic' to
'C:\Users\trevor\AppData\Local\Temp\SPCAF3fa77a\YourLicenseForFullFeatures.lic'
VERBOSE: Loading commandlet with following arguments:
VERBOSE: Path: 'C:\Users\trevor\AppData\Local\Temp\SPCAF3fa77a\spcaf.exe'
VERBOSE: InputFiles: 'C:\Users\trevor\desktop\Nauplius.SharePoint.FoundationSync.wsp'
VERBOSE: SettingsFile: 'null'
VERBOSE: Reports: 'DOCX'
VERBOSE: LicenseFiles: 'null'
VERBOSE: TempDirectory: 'null'
VERBOSE: OutputFile: 'C:\Users\trevor\desktop\FoundationSync.docx'
VERBOSE: Verbosity: 'normal'
VERBOSE: LogFile: 'null'
VERBOSE: SkipProjectCreation: 'False'
VERBOSE: Timeout: '2147483647'
VERBOSE: ------ SPCAF, Version: 5.2.3.3625 --------------------------------------
VERBOSE: Running SharePoint Code Analysis
VERBOSE: Authors: RENCORE AB
VERBOSE: Version: 5.2.3.3625
VERBOSE: ------ Running SharePoint Code Analysis with following parameters: -----
VERBOSE:  SourceFiles:  C:\Users\trevor\desktop\Nauplius.SharePoint.FoundationSync.wsp
VERBOSE:  SettingsFile:
VERBOSE:  OutputFile:   C:\Users\trevor\desktop\FoundationSync.docx
VERBOSE:  Reports:      DOCX
VERBOSE:  LogFile:      C:\Users\trevor\AppData\Local\SPCAF\SPCAF.log
VERBOSE: ------ SPCAF: Starting analysis please wait... -------------------------
VERBOSE: License: SPCAF Enterprise
VERBOSE: Arguments validation successfully
VERBOSE: Application Loaded
VERBOSE: Running Analysis on 24 elements...
VERBOSE: FoundationSync_Inventory.docx finished.
VERBOSE: FoundationSync_Dependencies.docx finished.
VERBOSE: FoundationSync_Metrics.docx finished.
VERBOSE: FoundationSync_MigrationAssessment.docx finished.
VERBOSE: FoundationSync_Rules.docx finished.
VERBOSE: Saving project successfully
VERBOSE: Found 6 critical errors, 64 errors, 18 critical warnings, 21 warnings, 199 metric information, 19 dependency
information, 19 inventory information, 5 files ignored
VERBOSE: ------ SharePoint Code Analysis completed -------------------------------------------
VERBOSE: Complete execution duration: 00:00:17.9148805
VERBOSE: Analysis Summary:
VERBOSE: Duration: '00:00:19.3438584'
VERBOSE: Violation Summary:
VERBOSE: CriticalErrors: '6'
VERBOSE: Errors: '64'
VERBOSE: CriticalWarnings: '18'
VERBOSE: Warnings: '21'
VERBOSE: Deleting temp folder 'C:\Users\trevor\AppData\Local\Temp\SPCAF3fa77a'


FatalErrorOccured : False
CriticalErrors    : 6
Errors            : 64
CriticalWarnings  : 18
Warnings          : 21
Inventories       : 19
Dependencies      : 19
Metrics           : 199
```

For the developer, Visual Studio has a similar toolset, but instead of examining a WSP, it will compile your code immediately before analysis and provide the output in the Error List. This is a much more convenient solution for developers that outputs to the Error List window.

![SPCAF_6](/assets/images/2014/12/SPCAF_6.png)

As with the desktop client, here you can also create a Dependency Graph (DGML) for your solution.

![SPCAF_7](/assets/images/2014/12/SPCAF_7.png)

Just like with the desktop client, reports are exportable to allow others to consume them! This is great, because this means not everyone requires a license to view the output of SPCAF.

![SPCAF_8](/assets/images/2014/12/SPCAF_8.png)

To provide a summary, RENCORE AB has provided 3 different methods of reviewing and evaluating code quality for developers, QA analysis, and IT Professionals. SPCAF has a trial license available which provides a good overview of the capabilities of the product. Microsoft MVPs, MCSMs/MCMs, and Microsoft FTEs may also request a SPCAF Enterprise license for free. I would highly recommend this from the IT Professional aspect simply because it may provide a way to keep code quality up and prevent lower quality solutions submitted by developers causing stability issues with a farm.
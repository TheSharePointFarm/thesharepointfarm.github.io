---
layout: post
title: InfoPath Forms Services Disallowed File Extensions is Hardcoded
tags: [sp2010]
---

The InfoPath Forms Service in SharePoint 2010 leverages a Web Application's `BlockedFileExtensions` property, but it also has it's own list of extensions which unfortunately cannot be overridden.

During the validation of a file type upload, the file first passes through the method which validates that the file type is not on the InfoPath file type block list.  See below for the list of file types that InfoPath disallows.

```csharp
private static bool IsInInfoPathBlockedExtensionList(string extension)
{
    if (_infoPathBlockedExtensionList.Count == 0)
    {
        lock (_infoPathBlockedExtensionList)
        {
            if (_infoPathBlockedExtensionList.Count == 0)
            {
                string str = "ADE,ADP,APP,ASP,BAS,BAT,CER,CHM,CMD,CNT,COM,CPL,CRT,CSH,EXE,FXP,GADGET,HLP,HPJ,HTA,INF,INS,ISP,ITS,JS,JSE,KSH,LNK,MAD,MAF,MAG,MAM,MAQ,MAR,MAS,MAT,MAU,MAV,MAW,MDA,MDB,MDE,MDT,MDW,MDZ,MSC,MSI,MSP,MST,OPS,PCD,PIF,PRF,PRG,PS1,PS1XML,PS2,PS2XML,PSC1,PSC2,PST,REG,SCF,SCR,SCT,SHB,SHS,TMP,URL,VB,VBE,VBS,VSMACROS,VSS,VST,VSW,WS,WSC,WSF,WSH";
                foreach (string str2 in str.Split(new char[] { ',' }))
                {
                    _infoPathBlockedExtensionList.Add(str2, str2);
                }
            }
        }
    }
    return _infoPathBlockedExtensionList.ContainsKey(extension.ToUpperInvariant());
}
```

After passing out of this method, it returns to a method which parses the Web Application's BlockedFileExtensions property:

```csharp
{...}
        SPWebApplication webApplication = SPContext.Current.Site.WebApplication;
        return ((webApplication != null) && webApplication.BlockedFileExtensions.Contains(str2));
```

If the file type is not in the InfoPath file type block list, but is in the BlockedFileExtensions property, this can be fixed by running the following PowerShell code from the SharePoint Management Shell:

```powershell
$wa = Get-SPWebApplication http://webappUrl
$wa.BlockedFileExtensions.Remove("ext")
$wa.Update()
```
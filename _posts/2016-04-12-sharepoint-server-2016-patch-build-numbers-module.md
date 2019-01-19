---
layout: post
title: SharePoint Server 2016 Patch Build Numbers Module
tags: [sp2016]
---

As the database schema is not generally being updated in SharePoint Server 2016, the farm build number may not increment as we're used to. Instead of relying on the farm build number, we'll need to start taking a look at individual patches to see what has been applied. I've created a PowerShell module that retrieves all of the SharePoint Server 2016 patch build numbers in an easy to consume format. It will tell you the product name, if the patch listed is the latest, as well as if there are any servers the patch is missing from.

Because the number of products in SharePoint Server 2016 has been reduced significantly, so the list won't be quite so long compared to past versions of SharePoint. This script is compatible with SharePoint Server 2013 and should be compatible with SharePoint Server 2010 as well.

Save the below script as `SPProductInformation.psm1`, then execute `Get-SPProductInformation`. Optionally pass a path to the cmdlet to save the output as CSV automatically, e.g. `Get-SPProductInformation C:\users\username\desktop`.  Let me know what you think, or if you have any ideas for improvements!

```powershell
Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0

function Get-SPProductInformation
{
    param
    (
     [string]
     [Parameter(Mandatory=$false)]
     $path = [string]::Empty
    )

    $patchList = @()
    $products = Get-SPProduct

    if($products.Count -lt 1)
    {
        Write-Host -ForegroundColor Red "No Products found."
        break
    }

    foreach($product in $products.PatchableUnitDisplayNames)
    {
        $unit = $products.GetPatchableUnitInfoByDisplayName($product)
        $i = 0

        foreach($patch in $unit.Patches)
        {
            $obj = [PSCustomObject]@{
                DisplayName = ''
                IsLatest = ''
                Patch = ''
                Version = ''
                SupportUrl = ''
                MissingFrom = ''
            }

            $obj.DisplayName = $unit.DisplayName

            if ($unit.LatestPatch.Version.ToString() -eq $unit.Patches[$i].Version.ToString())
            {
                $obj.IsLatest = "Yes"
            }
            else
            {
                $obj.IsLatest = "No"
            }
                        
            if (($unit.Patches[$i].PatchName) -ne [string]::Empty)
            {
                if ($unit.Patches[$i].ServersMissingThis.Count -ge 1)
                {
                    $missing = [System.String]::Join(',',$unit.Patches[$i].ServersMissingThis.ServerName)
                }
                else
                {
                    $missing = ''
                }

                $obj.Patch = $unit.Patches[$i].PatchName
                $obj.Version = $unit.Patches[$i].Version.ToString()
                $obj.SupportUrl = $unit.Patches[$i].Link.AbsoluteUri
                $obj.MissingFrom = $missing
                $missing = $null
            }
            else
            {
                $obj.Patch = "N/A"
                $obj.Version = "N/A"
                $obj.SupportUrl = "N/A"
                $obj.MissingFrom = "N/A"
            }

            $patchList += $obj
            $obj = $null
            ++$i
        }
    }

    if ($path -ne '')
    {
        try
        {
            Test-Path $path | out-null
        }
        catch
        {
            Write-host -ForegroundColor Red "Invalid path."
            break
        }
        $date = Get-Date -Format MM-dd
        $farm = Get-SPFarm

        if ($path.EndsWith('.csv'))
        {
            $patchList | Export-Csv $path -NoTypeInformation
             Write-Host -ForegroundColor Green "Build information exported to $path"
        }
        else
        {
            $patchList | Export-Csv "$path\$date-$($farm.EncodedFarmId).csv" -NoTypeInformation
            Write-Host -ForegroundColor Green "Build information exported to $path\$date-$($farm.EncodedFarmId).csv"
        }
    }
    else
    {
        return $patchList
    }
}
```
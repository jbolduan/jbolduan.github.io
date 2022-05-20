---
title: Automating Against the WinGet Database Using PowerShell
date: 2022-05-20 09:12:27 -0500
Categories: WinGet PowerShell Automation
---

[WinGet](https://docs.microsoft.com/en-us/windows/package-manager/winget/) is a tool which allows you to install, update, and remove software from a Windows computer.  It's built into the most recent versions of Windows 10 and Windows 11.  It's a fantastic tool for managing and updating the software on a singe device but not very useful for managing software on multiple devices or for detecting new versions of software and deploying through existing enterprise management tools.

Quick self promo all the code used as examples here has been integrated into our [UMN-AutoPackager repository](https://github.com/umn-microsoft-automation/UMN-AutoPackager).  This is a tool for automatically detecting, packaging and deploying applications into Microsoft ConfigMgr (SCCM/MECM/MEMCM/etc).

## The Executable

I started out my journey but simply using the winget executable.  This works for some things but the text output to the console is all you get back and fully automating against it is very complicated and will be subject to changes to how data is presented to end users.

```powershell
winget show Microsoft.Edge
```

Output:

```text
Found Microsoft Edge [Microsoft.Edge]
Version: 101.0.1210.47
Publisher: Microsoft Corporation
Publisher Url: https://www.microsoft.com/en-US/
Publisher Support Url: https://support.microsoft.com/en-us/microsoft-edge
Author: Microsoft Corporation
Moniker: microsoft-edge
Description: World-class performance with more privacy, more productivity, and more value while you browse.
Homepage: https://www.microsoft.com/en-us/edge
License: MIT License
License Url: https://www.microsoft.com/en-us/servicesagreement/
Privacy Url: https://privacy.microsoft.com/en-US/privacystatement
Copyright: Copyright (C) Microsoft Corporation
Copyright Url: https://www.microsoft.com/en-us/servicesagreement/
Installer:
  Type: wix
  Download Url: https://msedge.sf.dl.delivery.mp.microsoft.com/filestreamingservice/files/555154c3-778c-459a-b79b-24f5d3e2134f/MicrosoftEdgeEnterpriseX64.msi
  SHA256: 1cd499d7758e20e548c0a028ac7e8cadf5e35c851c9e960fffae8d7416d4dd1d
```

This example is probably the best formatted output available and it actually quite easy to pull information out of.  Others have many multiline items which wreck havoc on automation.  It was however easy to pull specific fields and automate against them such as Version: x.x.x.x.

There's a second major downside to automating using the winget executable which is it's not available for Windows Server.  If you want to have your automation running on a Windows Server (or better yet, a Windows Server in Azure Automation) you're out of luck.

## GitHub

This brought me to my next automation idea which is workable but has some limitation and that's going directly against the winget-pkgs GitHub repository.  It's the source that generates the backend info for packages available through winget.  It's a public repo so you can use Powershell to easily pull data out of it.  The only major downside is GitHub API rate-limiting.  I wasn't sure how many API calls we'd end up making in the end so chose to avoid this if possible.

## Backend

This led me to the final option and the one we went with.  Directly querying the winget backend.  I stumbled across [this stackoverflow post](https://stackoverflow.com/questions/61916158/structure-of-winget-source-repositories) which laid the groundwork for how the backend works.

The first piece you'll need is to pull down the `source.msix` file from the backend.

```powershell
Invoke-WebRequest -Uri "https://winget.azureedge.net/cache/source.msix" -OutFile "$Path\source.msix"
Expand-Archive -Path "$Path\source.msix" -DestinationPath "$Path\source" -Force
```

This will download the msix file and expand it into the same `$Path` directory.

The msix includes a `Public\index.db` file which is a SQLite database.  This is the file that contains all the information about the packages available.  It's structure is fairly simple and VERY easy to automate agains using the [PSSQLite](https://www.powershellgallery.com/packages/PSSQLite/) module from [@ramblingcookiemonster](https://twitter.com/pscookiemonster).

There are 20 tables in the database but the primary one we need is the manifests table.  This table contains the indexes for the other useful tables to build a list of all the applications available, their version, and the link to their yaml definition file we can download.

![SQLite DB Tables](/assets/images/2022-05-20-Automating-Against-the-WinGet-Database-Using-PowerShell/sqlite-db-tables.png)

To pull the initial information you'll need is a simple query

```sql
SELECT
    manifest.rowid,
    ids.id,
    names.name,
    monikers.moniker,
    versions.version,
    pathparts.pathpart,
    pathparts.parent
FROM 
    manifest
    LEFT JOIN ids ON ids.rowid=manifest.id
    LEFT JOIN names ON names.rowid=manifest.name
    LEFT JOIN monikers ON monikers.rowid=manifest.moniker
    LEFT JOIN versions ON versions.rowid=manifest.version
    LEFT JOIN pathparts ON pathparts.rowid=manifest.pathpart
```

This will give us the results with only one field (pathpart) needing more complicated handling.  Pathpart references the pathparts table, this table holds a piece of the path and the parent for that piece.  I'm sure there's a way to build the URL using SQL but I'm not that good at SQL so I leveraged PowerShell to do this.  First I needed a second query to pull all the pathparts from the database.

```sql
SELECT
    *
FROM
    pathparts
```

You can pull all this info into variables in PowerShell using the PSSQLite module:

```powershell
$AllDataQuery = @"
select manifest.rowid, ids.id, names.name, monikers.moniker, versions.version, pathparts.pathpart, pathparts.parent
from manifest
left join ids on ids.rowid=manifest.id
left join names on names.rowid=manifest.name
left join monikers on monikers.rowid=manifest.moniker
left join versions on versions.rowid=manifest.version
left join pathparts on pathparts.rowid=manifest.pathpart
"@

$RootQuery = "SELECT * FROM pathparts"

$AllApps = Invoke-SqliteQuery -DataSource $Database -Query $AllDataQuery
$AllPathparts = Invoke-SqliteQuery -DataSource $Database -Query $rootquery
```

Now that we have all the info from the database we'll need, we build lookup hashtables for pathparts.  This speeds up the process significantly.

```powershell
[hashtable]$parents = @{}
[hashtable]$pathparts = @{}
foreach ($pathpart in $AllPathparts) {
    $parents["$($pathpart.rowid)"] = $pathpart.parent
    $pathparts["$($pathpart.rowid)"] = $pathpart.pathpart
}
```

Now we just need to go through the applications, build the manifest path and an output object.

```powershell
[System.Collections.ArrayList]$OutputObject = @()
$OutputObject.Clear()
foreach ($App in $AllApps) {
    $AppManifestPath = $App.pathpart
    $PathBuilder = $App.parent
    do {
        $Parent = $parents["$PathBuilder"]
        $PathPart = $pathparts["$PathBuilder"]
        $AppManifestPath = "$PathPart" + "/" + $AppManifestPath
        $PathBuilder = $Parent
    } while ($null -ne $PathBuilder)

    $AppManifestPath = "https://winget.azureedge.net/cache/" + $AppManifestPath
    $null = $OutputObject.Add((New-Object PSObject -Property ([ordered]@{
                    RowId        = $App.rowid
                    Id           = $App.id
                    Name         = $App.name;
                    Moniker      = $App.moniker;
                    Version      = $App.version;
                    ManifestPath = $AppManifestPath;
                })))
}
```

`$OutputObject` will now contain all the information for every package currently available in the WinGet repository.  You can get the function below [Get-WinGetAllPackages](#Get-WinGetAllPackages) to use this output.

Once we have a list of all the applications we can filter down very easily to get just the application(s) we're looking for.  For example, if you want to filter on Id and return any fields that contain the word "VideoLAN.VLC" you could leverage the following code:

```powershell
$Id = "VideoLAN.VLC"
$FindPackages = Get-WinGetAllPackages -Path $Path
[System.Collections.ArrayList]$OutputObject = @()
$OutputObject.Clear()
foreach ($App in $FindPackages) {
    if($App.Id -like $Id) {
        $null = $OutputObject.Add($App)
    }
}
```

Now you have a list of all the packages that contains the VideoLAN.VLC Id.  This sometimes is okay but often it will return multiple values because older versions of a software application exist.  This can be fixed by searching for the latest.  We're going to use a Compare-Version function from the [UMN-AutoPackager repository](https://github.com/umn-microsoft-automation/UMN-AutoPackager).  I'll include the code for the [Compare-Version](#compare-version) function below.

```powershell
$LatestObject = $null

foreach ($Object in $OutputObject) {
    if ($null -eq $LatestObject) {
        $LatestObject = $Object
    }
    elseif (Compare-Version -ReferenceVersion "$($LatestObject.Version)" -DifferenceVersion "$($Object.Version)" -Verbose -InformationAction Continue) {
        $LatestObject = $Object
    }
}
```

Now the $LatestObject will contain the latest version of the package.  You can find the full code for the [Find-WinGetPackages](#find-wingetpackages) function below.

From here, it's simple to pull the manifest file from the database and convert it into an object which can be used in PowerShell using the [powershell-yaml](https://www.powershellgallery.com/packages/powershell-yaml/) module.

```powershell
$App = Find-WinGetPackages -Id "VideoLAN.VLC" -Latest

$YamlObject = [Test.Encoding]::UTF8.GetString((Invoke-WebReqeust -Uri $App.ManifestPath).Content) | ConvertFrom-Yaml
```

And BOOM, you've got all the information about a specific application from the winget repository without needing winget or anything other than PowerShell and a few PowerShell modules.

## Files

### Get-WinGetAllPackages

```powershell
function Get-WinGetAllPackages {
    [cmdletbinding()]
    param(
        [Parameter(Position = 0,
            ValueFromPipeline = $true,
            ValueFromPipelineByPropertyName = $true,
            HelpMessage = 'Path to store the WinGet source (default is ($env:TEMP\WinGetSource)).')]
        [Alias("PSPath")]
        [ValidateNotNullOrEmpty()]
        [string]
        $Path = "$($env:TEMP)\WinGetSource",

        [string]$DatabasePath = "$($env:TEMP)\WinGetSource\source\Public\index.db"

    )

    $AllDataQuery = @"
select manifest.rowid, ids.id, names.name, monikers.moniker, versions.version, pathparts.pathpart, pathparts.parent
from manifest
left join ids on ids.rowid=manifest.id
left join names on names.rowid=manifest.name
left join monikers on monikers.rowid=manifest.moniker
left join versions on versions.rowid=manifest.version
left join pathparts on pathparts.rowid=manifest.pathpart
"@

    $RootQuery = "SELECT * FROM pathparts"

    $AllApps = Invoke-SqliteQuery -DataSource $DatabasePath -Query $AllDataQuery
    $AllPathparts = Invoke-SqliteQuery -DataSource $DatabasePath -Query $rootquery

    [hashtable]$parents = @{}
    [hashtable]$pathparts = @{}
    foreach ($pathpart in $AllPathparts) {
        $parents["$($pathpart.rowid)"] = $pathpart.parent
        $pathparts["$($pathpart.rowid)"] = $pathpart.pathpart
    }

    [System.Collections.ArrayList]$OutputObject = @()
    $OutputObject.Clear()
    foreach ($App in $AllApps) {
        $AppManifestPath = $App.pathpart
        $PathBuilder = $App.parent
        do {
            $Parent = $parents["$PathBuilder"]
            $PathPart = $pathparts["$PathBuilder"]
            $AppManifestPath = "$PathPart" + "/" + $AppManifestPath
            $PathBuilder = $Parent
        } while ($null -ne $PathBuilder)

        $AppManifestPath = "https://winget.azureedge.net/cache/" + $AppManifestPath
        $null = $OutputObject.Add((New-Object PSObject -Property ([ordered]@{
                        RowId        = $App.rowid
                        Id           = $App.id
                        Name         = $App.name;
                        Moniker      = $App.moniker;
                        Version      = $App.version;
                        ManifestPath = $AppManifestPath;
                    })))
    }

    return $OutputObject
}
```

### Compare-Version

```powershell
<#
    .SYNOPSIS
        Takes a reference object and a difference object and determines if the reference object is greater than the difference object.
    .DESCRIPTION
        Takes a reference object and a difference object and determines if the reference object is greater than the difference object.

        Example: (reference) 1.0 > 0.1 (difference) would return true
    .EXAMPLE
        Compare-Version -ReferenceVersion "1.0.0.0" -DifferenceVersion "2.0.0.0" would return $false
    .EXAMPLE
        Compare-Version -ReferenceVersion "2.0.0-beta1" -DifferenceVersion "2.0.0-alpha12" would return $true
    .PARAMETER ReferenceVersion
        Version as a string which is on the left side of the greater than equation.
    .PARAMETER DifferenceVersion
        Version as a string which is on the right side of the greater than equation.
#>
function Compare-Version {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory = $true,
            HelpMessage = "Version as a string which is on the left side of the greater than equation.")]
        [ValidateNotNullOrEmpty()]
        [string]$ReferenceVersion,

        [Parameter(Mandatory = $true,
            HelpMessage = "Version as a string which is on the right side of the greater than equation.")]
        [ValidateNotNullOrEmpty()]
        [string]$DifferenceVersion
    )
    
    $SemVerRegex = "^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$"
    $SystemVersionRegex = "^(\*|\d+(\.\d+){0,3}(\.\*)?)$"
    $NumberOnlyRegex = "^[0-9]+$"

    if ((($ReferenceVersion -match $SemVerRegex) -and ($DifferenceVersion -match $SemVerRegex)) -or (($ReferenceVersion -match $NumberOnlyRegex) -and ($DifferenceVersion -match $NumberOnlyRegex))) {
        if ([System.Management.Automation.SemanticVersion]$ReferenceVersion -ge [System.Management.Automation.SemanticVersion]$DifferenceVersion) {
            return $false
        }
        else {
            return $true
        }
    }
    elseif (($ReferenceVersion -match $SystemVersionRegex) -and ($DifferenceVersion -match $SystemVersionRegex)) {
        if ([System.Version]$ReferenceVersion -ge [System.Version]$DifferenceVersion) {
            return $false
        }
        else {
            return $true
        }
    }
    else {
        throw "One or more input objects not a Semantic Version or System.Version"
    }
}
```

### Find-WinGetPackages

```powershell
function Find-WinGetPackages {
    [cmdletbinding()]
    param(
        [Parameter(Position = 0,
            ValueFromPipeline = $true,
            ValueFromPipelineByPropertyName = $true,
            HelpMessage = 'Path to store the WinGet source (default is ($env:TEMP\WinGetSource)).')]
        [Alias("PSPath")]
        [ValidateNotNullOrEmpty()]
        [string]
        $Path = "$($env:TEMP)\WinGetSource",

        [parameter(Mandatory = $true,
            ParameterSetName = "SearchById")]
        [string]$Id,

        [parameter(Mandatory = $true,
            ParameterSetName = "SearchByName")]
        [string]$Name,

        [parameter(Mandatory = $true,
            ParameterSetName = "SearchByMoniker")]
        [string]$Moniker,

        [parameter(Mandatory = $true,
            ParameterSetName = "SearchByPathPart")]
        [string]$PathPart,

        [switch]$Latest
    )

    if ($null -eq $WinGetPackages) {
        $FindPackages = Get-WinGetAllPackages -Path $Path -ForceUpdate:$ForceUpdate
    }
    else {
        $FindPackages = $WinGetPackages
    }

    [System.Collections.ArrayList]$OutputObject = @()
    $OutputObject.Clear()
    foreach ($App in $FindPackages) {
        if ($Id -ne '') {
            if ($App.Id -like $Id) {
                $null = $OutputObject.Add($App)
            }
        }

        if ($Name -ne '') {
            if ($App.Name -like $Name) {
                $null = $OutputObject.Add($App)
            }
        }

        if ($Moniker -ne '') {
            if ($App.Moniker -like $Moniker) {
                $null = $OutputObject.Add($App)
            }
        }

        if ($PathPart -ne '') {
            if ($App.Path -like $PathPart) {
                $null = $OutputObject.Add($App)
            }
        }
    }

    $LatestObject = $null

    if ($Latest) {
        foreach ($Object in $OutputObject) {
            if ($null -eq $LatestObject) {
                $LatestObject = $Object
            }
            elseif (Compare-Version -ReferenceVersion "$($LatestObject.Version)" -DifferenceVersion "$($Object.Version)" -Verbose -InformationAction Continue) {
                $LatestObject = $Object
            }
        }

        return $LatestObject
    }

    return $OutputObject
}
```

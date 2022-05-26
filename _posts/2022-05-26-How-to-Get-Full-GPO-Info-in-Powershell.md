---
title: How to Get Full GPO Info in Powershell
date: 2022-05-26 08:55:57 -0500
Categories: powershell gpo ad
---

A quick tip today, how to pull the GPO report info quickly into a variable.

```powershell
$GPOReport = ([xml](Get-GPOReport -Name "MyGPO" -ReportType xml)).GPO
```

This will pull the GPO report as an XML object and then pull the GPO portion of the returned object.  This is the part you'll usually want anyways and doing it in one line will pull all the relevant information.

It will let you pull the all the information including the current links of the GPO and going through the Computer/User policies.

```powershell
$GPOReport.LinksTo

$GPOReport.User.ExtensionData

$GPOReport.Computer.ExtensionData
```

Let's say you wanted to pull the restricted groups you could use the following:

```powershell
$GPOReport.Computer.ExtensionData.Extension.RestrictedGroups
```

You can explore all the various parts of the GPO by going one level deeper and piping the object into Get-Member (gm) to figure out what properties are available to explore further.

```powershell
$GPOReport.Computer.ExtensionData.Extension | Get-Member
```

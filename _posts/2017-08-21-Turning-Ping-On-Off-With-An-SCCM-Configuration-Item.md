---
title:  "Turning Ping On/Off With An SCCM Configuration Item"
date:   2017-08-21 17:59:34 +0000
categories: powershell memcm sccm mecm ping configuration CI
---
## Details

The best way I've found to disable ping using a configuration item through a
script is using a .Net class.  Our fleet is Windows 7+ and Server 2003(ick)+ so
the solution needs to be more robust than using the Server 2012+ built-in
firewall cmdlets.

The scripts are quite simple but we rely heavily on the following core code:

```powershell
# Get the firewall manager object from .Net

$Firewall = New-Object -ComObject HNetCfg.FwMgr

# Get the domain policy (0)

$Policy = $Firewall.LocalPolicy.GetProfileByType(0)

# Get the settings for ping

$IcmpSettings = $Policy.IcmpSettings
```

This code gets the firewall manager object out of .Net and then grabs the domain
Firewall policy then the ICMP settings for the domain policy.  This is flexible
so the .Net object should exist in older versions of Windows.

Now, there is really only one setting we care about in the ICMP Settings:

```powershell
$IcmpSettings.AllowInboundEchoRequest
```

If it's ```$true``` then ping is turned on, if it's ```$false``` then ping is disabled.

Thus, the discovery script is fairly simple, all we have to do is ```.ToString()```
the ```$IcmpSettings.AllowInboundEchoRequest``` and if we want to enable/disable
ping all we have to do is add an ```= $true``` or ```= $false``` to the statement.

Below I've put the full scripts I used for our code.

## Scripts<

### Discovery

```powershell
$Firewall = New-Object -ComObject HNetCfg.FwMgr

$Policy = $Firewall.LocalPolicy.GetProfileByType(0)

$IcmpSettings = $Policy.IcmpSettings

$IcmpSettings.AllowInboundEchoRequest.ToString()
```

### Remediation - Turn Ping On

```powershell
$Firewall = New-Object -ComObject HNetCfg.FwMgr

$Policy = $Firewall.LocalPolicy.GetProfileByType(0)

$IcmpSettings = $Policy.IcmpSettings

$IcmpSettings.AllowInboundEchoRequest = $true
```

### Remediation - Turn Ping Off

```powershell
$Firewall = New-Object -ComObject HNetCfg.FwMgr

$Policy = $Firewall.LocalPolicy.GetProfileByType(0)

$IcmpSettings = $Policy.IcmpSettings

$IcmpSettings.AllowInboundEchoRequest = $false
```

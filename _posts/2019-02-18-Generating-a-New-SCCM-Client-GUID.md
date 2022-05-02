---
title:  "Generating a New SCCM Client GUID"
date:   2019-02-18 16:10:11 +0000
categories: sccm client guid
---
- Stop the SCCM service in Powershell using `Stop-Service ccmexec` and then wait for it to fully stop.
- Rename the `C:\Windows\SMSCFG.INI` file to something like `C:\Windows\SMSCFG.old.INI`
- Force the computer to update it's AD certificate:

```powershell
$certs = Get-Certificate -CertStoreLocation Cert:\\LocalMachine\\My -Template Machine 

$thumbprint = $certs.Certificate.Thumbprint certreq.exe -enroll -q -machine -cert "\*$thumbprint\*" Renew
```

- Delete the computer object out of SCCM.
- Restart the SCCM service using `Start-Service ccmexec` and then it should start up, generate a new GUID and re-create it's object in SCCM with the new GUID.
- Run **Machine Policy Retreval & Evaluation Cycle** in the SCCM client Control Panel.

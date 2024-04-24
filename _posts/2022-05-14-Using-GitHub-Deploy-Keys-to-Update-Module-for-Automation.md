---
title: Using GitHub Deploy Keys to Update Module for Automation
date: 2024-04-24 10:56:00 -0500
Categories: Automation Powershell
---

1. Generate SSH key
2. Copy to server (e.g. C:\.ssh)
3. Ensure ssh-agent is set Automatic (Delayed Start)

    ```powershell
    Get-Service ssh-agent | Select-Object -Property StartType

    StartType
    ---------
    Disabled
    ```

    ```powershell
    Get-Service ssh-agent | Set-Service -StartupType Automatic
    ```

    ```powershell
    Get-Service ssh-agent

    Status   Name               DisplayName
    ------   ----               -----------
    Running  ssh-agent          OpenSSH Authentication Agent
    ```

4. Set ACL's on the ssh key directory

    ```powershell
    $acl = get-acl .\.ssh\

    $accessrule = new-object System.Security.AccessControl.FileSystemAccessRule("BUILTIN\Administrators","FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")

    $acl.SetAccessRuleProtection($true, $false)

    Set-Acl .\.ssh\ $acl
    ```

5. Add the ssh key (you'll need the passphrase you set up earlier)

    ```shell
    ssh-add {path to key}
    ```

6. Install Git

    - If you're using a headless system chocolatey can help keep chocolatey up to date until winget is supported on server operating systems.

7. Ensure your key still exists (just in case)

    ```shell
    ssh-add -l
    ```

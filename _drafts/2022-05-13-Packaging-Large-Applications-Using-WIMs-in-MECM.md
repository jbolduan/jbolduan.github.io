---
title:  "Packaging Large Applications Using WIMs in MECM"
date: 2022-05-13 12:22:29 -0500
categories: MECM Packaging WIM
---

1. Run Powershell as Administrator
2. Make sure files are local (I've encountered issues with files on remote file shares)
3. Turn the directory into a WIM

    ```powershell
    New-WindowsImage -ImagePath "{pathToSaveWIM}\{Vendor}-{AppName}-{version}.wim" -CapturePath "{directoryToMakeAWIM}" -Name "{Vendor}{AppName}InstallerFiles"
    ```

4. Install Script Logic
5. Uninstall Script Logic

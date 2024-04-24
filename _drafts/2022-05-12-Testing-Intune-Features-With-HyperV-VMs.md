---
title:  "Testing Intune Features With Hyper-V VMs"
date:   2022-05-12 08:54:00 -0500
categories: Intune HyperV testing iso
---

## Prerequisites

- Enable HyperV on the machine that will be used for testing

## Process

1. Create a VM in HyperV
2. Create Enrollment Package using Windows Imaging and Configuration Designer
3. Turn enrollment package into ISO file.

    ```powershell
    New-ISOFile -Source 'C:\Users\{username}\Documents\Windows Imaging and Configuration Designer (WICD)\Enrollment' -DestinationISO C:\Temp\Enrollment.iso -Title 'BulkEnrollment'
    ```

## Resources

- [New-ISOFile.ps1](https://github.com/TheDotSource/New-ISOFile/blob/main/New-ISOFile.ps1)

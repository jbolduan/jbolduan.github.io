---
title:  "Using VNC For Remote Imaging in SCCM Task Sequences"
date:   2019-05-20 15:16:09 +0000
categories: sccm task sequence OSD VNC remote control
---
I want to quickly thank Gary Blok as he was the inspiration for the process we're currently using in his [DaRT & VNC Remote during OSD without Integration](https://garytown.com/dart-vnc-remote-during-osd-without-integration) post. If you'd like to leverage DaRT in WinPE I would strongly encourage looking through that blog post for additional information.

We made a choice to go pure VNC as it allowed for a consistent process through both the WinPE and Windows phases of imaging. DaRT only works while you're in WinPE and will not in Windows so if you wanted to have remote capabilities through the whole task sequence you'd still need to build VNC for Windows.

The huge advantage of using a VNC executable in a package is that it's a transient item that is gone after a reboot. It adds more steps to your task sequence but makes sure there's nothing left over after the task sequence has run through.

## Obtain VNC

The process is fairly simple, first you need to get the latest version of the [Ultra VNC Server](https://www.uvnc.com). From the downloads page make sure to get the **bin zip ALL**. You'll also probably want to grab the latest MSI to package and deploy as a viewer for those doing the imaging.

## Setup Server

For the server package, grab the winvnc.exe file out of zip for the bit level of your boot media/OS. Likely this is 64 but if you have special circumstances you may need to build a 64 and 32 bit version of the package.

Once you've moved the winvnc.exe file to a new directory, run it as an administrator and configure the settings. You'll want to setup a good password so that it cannot be easily cracked. Also, configure the security settings to conform to the security standards of your environment. Once you have everything configured how you like it click apply and it should generate an UltraVNC.ini file in the same directory. If you'd like to manually configure this file check out more info in the [UltraVNC docs](http://www.uvnc.com/docs/uvnc-server/69-ultravncini.html).

## Setup Powershell Launch File - WinPE

I use a Powershell file to launch the VNC Server as it lets me write a configuration file off to a file server with a super secure password in it. You could leverage the password generator for VNC to make a unique password for each device but that is a step I haven't gone down yet.

Here is what the Powershell should look like:

```powershell
$VNCConnection = @"
[connection]
host={HOST_IP}

"@
$TS = New-Object -ComObject Microsoft.SMS.TSEnvironment

$IPAddress = (Get-WmiObject win32_Networkadapterconfiguration | Where-Object{ $_.ipaddress -notlike $null }).IPaddress | Select-Object -First 1
$FileContents = $VNCConnection.Replace("{HOST_IP}", $IPAddress)

$ComputerName = $TS.Value('_SMSTSMachineName')

wpeutil DisableFirewall

cmd.exe /c start winvnc.exe

if($null -ne $ComputerName) {
    New-Item -Path "Z:\" -Name "$ComputerName.vnc" -ItemType File -Value $FileContents -Force
} else {
    New-Item -Path "Z:\" -Name "$IPAddress.vnc" -ItemType File -Value $FileContents -Force
}
```

Firstly, update $VNCConnection to be the contents of your ini file if you're looking to leverage the ability to write a VNC shortcut to a file share. You'll want to set up the host={HOST_IP} like I did as this is how we make sure the host is binding to the correct IP address.

The script will grab the first active network IP address. I have yet to run into any issues with this but I imagine you could if you have multiple network adapters and some don't allow VNC traffic so your mileage may very on this method.

It also needs to disable the WinPE firewall. I didn't run into a good way to allow just VNC traffic through so this could easily be a deal breaker for some organizations. If you find a good way to punch a whole through the firewall let me know in the comments or on Twitter!

It starts VNC and then generates the shortcut we previously discussed. I use the built in map network drive function of the task sequence.

## Setup Powershell Launch File - Windows

Next we have to build the Powershell script for launching the server once the task sequence is running from Windows.

```powershell
$FileContents = @"
[connection]
host={HOST_IP}

@"

$TS = New-Object -ComObject Microsoft.SMS.TSEnvironment

$IPAddress = $TS.Value("IPAddress001")
$FileContents = $FileContents -replace "{HOST_IP}", $IPAddress

$ComputerName = $TS.Value('_SMSTSMachineName')

netsh advfirewall firewall add rule name="VNC OSD 5900" protocol=tcp dir=in localport=5900 action=allow

cmd.exe /c start winvnc.exe

if($null -ne $ComputerName) {
    New-Item -Path "Z:\" -Name "$ComputerName.vnc" -ItemType File -Value $FileContents
} else {
    New-Item -Path "Z:\" -Name "$IPAddress.vnc" -ItemType File -Value $FileContents
}
```

This script is virtually identical but this time we open a specific firewall port. In this example I set it up for the default port of 5900 but you'll want to set it to whatever you've configured in your settings and once again you'll want to be sure you have all the settings in your ini in the $FileContents variable.

## Adding Steps to Task Sequence

Now that we have the executable, configuration and launch scripts we are ready to build the steps in the task sequence. To make it easier to execute I make a group that I can copy paste where I need it. This could possibly be made a child task sequence as well.

First, connect to the network share using the **Connect to Network Folder** task sequence step. Give it the user account to map the drive as and define a drive letter. I used Z:\\ for simplicity.

Second, create a **Run PowerShell Script** step. Set the package source for the script to the package of files created earlier and put the script name in the **Script name** field.

Lastly, disconnect the network drive. This will make sure if you have other steps leveraging the same network resource with different credentials don't collide with this step.

You'll need to setup this configuration for both the WinPE script and the Windows script. Make sure to put this block of steps after every reboot or where a reboot occurs naturally in the task sequence and it will pull up VNC and run it from the package. After a reboot it should all be gone.

## Cleanup

At the end of the task sequence, make sure you're removing the firewall rule we created in the script using the following **Run Command Line** step in the task sequence. I just put the code directly in the **command line** section of the step.

```cmd
netsh advfirewall firewall delete rule name="VNC OSD 5900"
```

You may also want to make sure you have a post action to reboot the computer. This can be done by setting up a **Set Task Sequence Variable** step and setting the **SMSTSPostaction** variable to be:

```cmd
cmd /c shutdown /r /t 300 /f
```

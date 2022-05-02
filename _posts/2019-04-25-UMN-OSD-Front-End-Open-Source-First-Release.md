---
title:  "UMN OSD Front End Open Source First Release"
date:   2019-04-25 18:03:06 +0000
categories: sccm frontend C# release UMN OSD Front End
---
## Overview

We've [built an OSD front end](https://github.com/umn-microsoft-automation/UMN-OSDFrontEnd/releases/tag/v1.0.0.0) that works well for our uses and may serve other parts of the SCCM community as well. There are some features that we would like to implement which are currently missing such as setting a bind location but this isn't something we currently do in our task sequences and so it wasn't an immediate priority. The front end is broken down into five tabs pre-flight, computer name, backup options, user profiles, and applications.

## Pre-Flight

![pre flight screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/pre_flight_screen.png)

Example pre-flight checks from a re-image task sequence.

We have built into the front end logic for a few different types of checks so far:

- Physical Disk Count
    - You determine how many disks can be detected and then as long as the number of physical disks is below that number it will return a pass.
- Ethernet Connection
    - Will check and pass if there is an active Ethernet connection.
- Network Connectivity Check
    - Will check (by ping) if a network location is online and return a pass if it is. I'm using this to ping a storage appliance just to confirm basic connectivity but this could ping all sorts of things.
- 64-Bit OS
    - Will check if the OS the front end is running on is 64-bit and then return a pass or fail based on if you've set the checkPassState to true (Pass if 64) or false (pass if not 64).
- Offline Files Detected
    - This one is still in progress to some degree. Currently, if you use this check it's going to go through the Win32\_UserProfile and look for RoamingConfigured. I would like to expand on this more but this was a short term check just to see how useful this would be in our environment.

## Computer Name

![computer name screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/computer_name_screen.png)

Example computer name tab.

The computer name tab is just a simple computer naming script essentially with some added checks on minimum and maximum length as well as starting and ending strings. All of these settings can be configured in the AppSettings.json file. You can also require or not require the various checks which will enable/disable the next button until the conditions are met.

## Backup Options

![backup options screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/backup_options_screen.png)

Backup options we use in our task sequence. Right now these are hard coded in but can/should be made more flexible in future versions of the tool.

WE only leverage two backup options and as of right now I've hard coded these into the form. We have a USMT option as well as a full disk backup to a WIM file. Both of these options set an OSD Variable:

- Wim backups set the OSDWIMBackup to True or False depending on checked state.
- USMT backups set the OSDUSMTBackup to True or False depending on checked state.

If you're not planning on implementing these I would disable this tab. A future version will include further options to customize this tab and possibly additional features if they're requested for our own internal task sequence.

## User Profiles

![user profiles screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/user_profiles_screen.png)

Example of the user profiles tab with a profile selected in the box but the name has been blocked out.

The user profiles tab was created when we started to do full disk images as backups during a re-image scenario. We wanted to cut down on the size of the images as much as possible and so removing profiles was the best option. You'll find that this interface will show every profile (excluding the logged on user) and you can select as many profiles as you like. Clicking the set profiles for deletion button will set a variable in the application that upon completion of the form will do the profile removals. This cadence was done because we experienced long delay's when trying to remove user profiles at this stage and it was a better user experience to move the deletion to the end of the form.

## Applications

![applications screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/applications_screen.png)

Example of applications list.

Now we come to my favorite tab. It was very much inspired by the [SCConfigMgr OSD FrontEnd](https://www.scconfigmgr.com/configmgr-osd-frontend/) which we would likely have used internally had we been able to open the source code and make minor modifications to fit our needs. I would very much encourage everyone to check out their front end as it's more feature complete than ours and may be a better solution depending on your needs.

Like their front end I'm using their [web service](https://www.scconfigmgr.com/configmgr-webservice/) to connect with SCCM and pull application names out of SCCM based on pre-applied administrative categories and then the checked applications are installed during a dynamic application install step.

![sccm install checked apps](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/sccm_install_checked_apps.png)

Step in task sequence to install applications.

The list can grow quite large but we've had reasonably good success with this even installing large numbers of big applications. Cadence could be an issue but we haven't encountered that so far either.

![finish screen](/assets/images/2019-04-25-UMN-OSD-Front-End-Open-Source-First-Release/finish_screen.png)

As you finish tabs they should be enabled and allow you to go back and re-do sections if needed. Once complete you just hit the complete button and it does everything it needs to do. If you exit the app using the close window button it will exit in error with code 22.

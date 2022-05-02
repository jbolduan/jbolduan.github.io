---
title:  "Building an OSD Front End Using C#: Part 1 Getting Started"
date:   2019-04-26 13:00:58 +0000
categories: sccm frontend C#
---
This is a blog series walking through how to build your own OSD Front End using C#.  It walks through everything I went through creating the [UMN-OSDFrontEnd](https://github.com/umn-microsoft-automation/UMN-OSDFrontEnd) project which is now open source and available on GitHub. I'm using Visual Studio.  If you are starting out it's the easiest (in my opinion) way to get up and running with C#.  There are other ways to build C# projects but I'm going to assume for the purposes of this guide that you have access to Visual Studio. You'll want to create a new WPF App (.NET Framework) project.  A key thing is to ensure you change your project's target framework from the latest version to .NET Framework 4.5.  This will ensure you're building to be compatible with WinPE.

![osd front end project creation](/assets/images/2019-04-26-Building-an-OSD-Front-End-Using-CSharp-Part-1/osdfrontendprojectcreation.png)

Next we will need to get all the NuGet packages we're going to use throughout the project by going to Project -> Manage NuGet Packages then search for these under the browse section:

- [ControlzEx](https://github.com/ControlzEx/ControlzEx)
    - The only reason this needs to be installed is that it's a requirement of the MahApps.Metro package.
- [MahApps.Metro](https://github.com/MahApps/MahApps.Metro)
    - This gives you access to a metro styled app using WPF.  It makes it way easier to customize the application and make it look good.
- [MahApps.Metro.Resources](https://github.com/MahApps/MahApps.Metro.Resources)
    - Requires MahApps.Metro
    - This contains a bunch of useful resources such as icons/images.
- [Mono.Options](https://github.com/xamarin/XamarinComponents/tree/master/XPlat/Mono.Options)
    - Gives us an easy way to parse parameters passed to the executable.
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
    - Makes it easier to handle json data.

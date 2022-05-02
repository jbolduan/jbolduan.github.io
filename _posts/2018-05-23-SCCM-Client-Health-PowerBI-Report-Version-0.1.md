---
title:  "SCCM Client Health PowerBI Report Version 0.1"
date:   2018-05-23 21:57:03 +0000
categories: sccm client health powerbi
---
Recently we've been struggling to get our client health under control. We implemented the script developed by [Anders Rodland](https://www.andersrodland.com/configmgr-client-health/) which has helped us fix large numbers of broken clients in our environment. I didn't really want to stand up an SSRS server just to report on client health so I took a stab and using PowerBI.

Installation of the report is very easy.  Simply download the PowerBI template [here](/assets/files/clienthealthscript_v0-1.zip) and open it in [PowerBI Desktop](https://powerbi.microsoft.com/en-us/desktop/).  On first running the script you'll be asked to enter in the SQL server hosting your script data, the database on that server and the clients table.  This should allow you to connect this easily in your environment even if you did some customization to how you're storing the SQL data.

![client health install](/assets/images/2018-05-23-SCCM-Client-Health-PowerBI-Report-Version-0.1/clienthealth_install.png)

The default (first) tab of the report is just a simple dashboard.  I was looking to track all the basics pulled in from the client health script so I could at a glance see where we might have a problem.  Below is a snapshot demonstrating how it breaks down each and shows where you might be encountering problems in your environment.

Additionally I was looking to pull in info about OS version and system architecture just to get a better understanding if any outliers in those areas were having special problems.

![client health dashboard](/assets/images/2018-05-23-SCCM-Client-Health-PowerBI-Report-Version-0.1/clienthealth_dashboard.png)

The second page of the report is an overview of what I was looking for in timeline data.  It shows when the script is doing client installs and how many were being installed as well as OS Updates data and Hardware Inventory data.  I added slicers (the sliders below each timeline) so you can zoom in your data to specific time periods if you'd like to see which machines did something in a specific time.  I've already found use in this looking for timeline outliers which are pulling my graphs way off track.

![client health date slicing](/assets/images/2018-05-23-SCCM-Client-Health-PowerBI-Report-Version-0.1/clienthealth_dateslicing.png)

The third page gives insights into client version including a gauge which can be configured to display how many up to date clients are in your environment.  It also includes breakdowns of last updated and last rebooted as well as operating system breakdowns.

![client health client versions insights](/assets/images/2018-05-23-SCCM-Client-Health-PowerBI-Report-Version-0.1/clienthealth_clientversionsinsights.png)

If there's any additional features to the report you think would be handy let me know on twitter [@jbolduan](https://twitter.com/jbolduan/) and I'll work on including them in future releases of the PowerBI template.

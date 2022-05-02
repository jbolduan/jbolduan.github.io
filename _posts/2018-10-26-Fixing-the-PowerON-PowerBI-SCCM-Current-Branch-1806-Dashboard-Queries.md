---
title:  "Fixing the PowerON PowerBI SCCM Current Branch 1806 Dashboard Queries"
date:   2018-10-26 17:54:27 +0000
categories: sccm current branch PowerON PowerBI dashboard
---
In our environment we're seeing failures on some of the queries because we have multiple devices coming back when there should only be a single record.  I've [created a gist](https://gist.github.com/jbolduan/292687ec88257507e4ee0184b2a118fa) with the updated SQL queries all you have to do is open up the corresponding queries and editing the root query running in SCCM.  Let me know if you find any additional edge cases where there could be potential fixes needed.

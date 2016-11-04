---
layout: post
title: "Setting Up PowerShell for Office 365"
subtitle: "Getting the plumbing in order for successfully managing your tenant"
date: 2016-11-03
category: Administration
tags: PowerShell
author: Eric
---

So you have a shiny new Offce 365 tenant. Now what? If you are anything like me, the first thing you want to do is to start plugging away from the PowerShell to administer your tenant. But how do you do that?

In this post, I am going to go through the process of getting your environment set up to start using PowerShell. This is focused more on the SharePoint side of the Office 365 suite.

##Prerequisites
In order to use PowerShell to talk to Office 365, you need PowerShell 3.0. I'm operating under the assumption that most of us in the business world are ssstill running Windows 7. It has PowerShell 2.0 installed. What do we do?
We install the Windows Management Framework to upgrade of course.
###Windows Management Framework
The [Windows Management Framework](http://www.microsoft.com/en-us/download/details.aspx?id=40855 "Download from Microsoft") contains updates to the client and ISE. Download and install these components.
###SharePoint Online Management Shell
The next thing we need to install is the [SharePoint Online Management Shell](http://www.microsoft.com/en-us/download/details.aspx?id=35588 "Download from Microsoft"). This contains all the DLLs needed to do our work.
###SharePoint Online Client Components
The next required component is the [SharePoint Online Client Components SDK](http://www.microsoft.com/en-us/download/details.aspx?id=42038 "Download from Microsoft"). These are all the various DLLs that we need to use CSOM in PowerShell. 
The SharePoint Online Client Components have a very limited set of commandlets at our disposal if you have ever worked with an on premesis installation of SharePoint. We need these client DLLs to be able to create our own functions
that bridge the gap. Once these are installed
##Optional Module, PnP PowerShell
If creating a slew of functions is not up your alley, there is a great publicly available project on GitHub called [Office 365 Dev PnP PowerShell CmdLets](https://github.com/OfficeDev/PnP-PowerShell). I won't be covering this in great detail. By the time I knew it was
out there, I already had a module of functions in the area of 3000 lines of code. I have used it for inspiration for gaps in our module.
##Other Module of Note
Depending on what workloads you are going to work with in Office 365, there are other PowerShell Modules you may want to install. They include, and are not limited to: 
 + [Azure Active Directory PowerShell Module](https://msdn.microsoft.com/en-us/library/azure/jj151815(v=azure.98).aspx "Download from Microsoft")
 + [Skype for Business PowerShell Module](https://www.microsoft.com/en-us/download/details.aspx?id=39366 "Download from Microsoft")
 + Exchange Online uses a different mechanism to get the required modules. They are pulled down from the server each time you connect.
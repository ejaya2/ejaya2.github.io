---
layout: post
title: "Integrating Google Analytics Into SharePoint Online"
subtitle: "Get deep analytics and page views in SharePoint Online with Google Analytics"
date: 2017-01-21
category: Administration
tags: [Auditing,PowerBI,Analytics]
author: Eric
---

Welcome back friends. Today in this catchy title, I'm going to go into detail on you you can get deep analytics on your SharePoint Online usage behavior by using User Custom Actions, Google Analytics, and Power BI. This has been something I wanted to blog about for a while. We have this setup running on a couple of our site collections that needed some rich analytics and metrics on user behavior. I've been putting it off mainly because of how long it will be, so if you find this interesting, I would appreciate some tweets.

There are a few pieces to pull this all together, but is very straight forward and easy to do. It will require some PowerShell to pull off as Microsoft has not given us an interface to manage User Custom Actions at this point.

## Pitfalls of the current state

### SharePoint itself

First, a little ground setting about the current state of analytics in SharePoint Online. One of the first things I noticed when we migrated up, was the dumbed down audit settings. If you go into Site Settings and go to Site Collection audit settings, you'll see a glaring omission from an on-premises installation, Viewing items has disappeared.

![Site Collection Audit Settings](http://ericjalexander.com/img/audit_settings.png "Site Collection Audit Settings") 

### Security and Compliance Center

The other place you might look is the Security and Compliance center within your Office Admin Portal. There is a lot of good data here, but unfortunately nothing that tracks page views. 

![Security and Compliance audit log search](http://ericjalexander.com/img/security_compliance.png "Security and Compliance audit log search") 

### Office 365 Adoption Content Pack Preview

The Office 365 team is working on a content pack for Power BI that aggregates a lot of data and information about the usage statistics of your tenancy. This is things like Yammer threads, Skype for Business chats and voice calls, and OneDrive for Business storage information. In the initial release of this content pack, I didn't see the specific SharePoint information I was looking/hoping for. So where does this leave us? I opted for the ever trusty, [http://google.com/analytics/](Google Analytics).

## Get a Google Analytics tracking code

So we've decided on Google Analytics, how do we get setup? Well, first you'll need a Google account. Go thorough the process of getting a tracking code by following the information provided by [Google](https://support.google.com/analytics/answer/1008015?hl=en). Save this analytics code as ga.js and upload it to the Site Assets or Style Library of your SharePoint site.

## Stapling your code to your site

Now that we have an analytics tracking code and have it saved in our site, it is time to wire it up. We do that with a user custom action. [User Custom Actions](https://msdn.microsoft.com/en-us/library/office/ee538686(v=office.14).aspx) are a seemingly little known component that has been around in SharePoint since the 2010 version. User Custom Actions allow you to staple your own custom JavaScript files to your sites, and when used with the Alternate CSS, there is absoloutely no reason to modify your master pages.

As part of our custom PowerShell library for SharePoint Online, I built a little function to create a new custom action to a site or web. That function looks like this:

```PowerShell
Function New-CustomAction{
    <#
		.SYNOPSIS
            Creates a new custom action on the spicifed address
	    .DESCRIPTION	
            Creates a new custom action on the spicifed address
	    .PARAMETER Url
            The Url to add a custom action to
        .PARAMETER ActionType
            Site or web UserCustomActions to find
        .PARAMETER Title
            The title of the CustomAction
        .PARAMETER ScriptSrc
            The full URL to the path of the file to attach
        .PARAMETER Sequence
            The sequence number in which this is to be executed, lower numbers get executed first
        .PARAMETER Description
            A description of the custom action
        .EXAMPLE
            Creates a UserCustomAction with the specified parameters
            New-CustomAction -Url "https://tenant.sharepoint.com/teams/eric/subsite" -ActionType "Web" -Title "Google Analytics Tracking code" -ScriptSrc "https://tenant.sharepoint.com/teams/eric/subsite/SiteAssets/ga.js" -Sequence 0
		.NOTES
        .LINK
        https://msdn.microsoft.com/en-us/library/office/dn531432.aspx#bk_UserCustomAction
        	
	#>
    Param(
        [Parameter(Mandatory=$true,HelpMessage="The URL of the site or web")][ValidateNotNull()]
        [string]$Url,
        [Parameter(Mandatory=$true,HelpMessage="The action type scope")][ValidateSet("Site","Web")]
        [string]$ActionType,
        [Parameter(Mandatory=$true,HelpMessage="The title of the CustomAction")][ValidateNotNull()]
        [string]$Title,
        [Parameter(Mandatory=$true,HelpMessage="The full URL to the path of the file to attach")][ValidateNotNull()]
        [string]$ScriptSrc,
        [Parameter(Mandatory=$true,HelpMessage="The sequence number in which this is to be executed")][ValidateNotNull()]
        [int]$Sequence,
        [Parameter(Mandatory=$false,HelpMessage="A description of the custom action")]
        [string]$Description
    )
    Begin{
        $context = New-Object Microsoft.SharePoint.Client.ClientContext($Url) 
        $context.Credentials = New-Object Microsoft.SharePoint.Client.SharePointOnlineCredentials($global:cloudAdminUser.UserName, $global:cloudAdminUser.Password)
                
        if($ActionType -eq "Site"){
            $site = $context.Site
        }
        else{
            $site = $context.Web
        }
        $context.Load($site)
        $context.ExecuteQuery()
    }
    Process{       
        $action = $site.UserCustomActions.Add()
        $action.Location = "ScriptLink"
        $action.ScriptSrc = $ScriptSrc
        $action.Title = $Title
        $action.Sequence = $Sequence
        if($Description){
            $action.Description = $Description
        }
        try{
            $action.Update()
            $context.ExecuteQuery()
            write-host -ForegroundColor Green "New UserCustomAction created successfully!" 
        }
        catch{
            Write-Host -ForegroundColor Red $_.Exception.Message
        }
    }
    End{
        $context.Dispose()
    }
}
```

To apply the custom action to a site collection, you would run a command like:

```
New-CustomAction -Url "https://tenant.sharepoint.com/teams/eric" -ActionType "Site" -Title "Google Analytics Tracking code" -ScriptSrc "https://tenant.sharepoint.com/teams/eric/SiteAssets/testscript.js?rev=1" -Sequence 0
```

With JavaScript files being heavily cached by browsers, I recommend adding a `?rev=1` query string parameter onto the end of the ScriptSrc. If at any time you need to update the JavaScript file, you simply update the ScriptSrc parameter to rev=2 and browsers will pull a fresh copy.

## Surfacing the data with Power BI

After applying the tracking code, open your Google Analytics dashboard and check the Real Time Overview. This is updating in real time to show traffic. Try browsing around your site and you should see hits start popping. The toughest part now is waiting a few days to get the data aggregation going.

Fast forward a few days and we're now ready to start getting some charts. Time to click that Power BI icon in your waffle, I mean App Launcher. I do want to point out, that what we are going to do will be free. There are other ways to do this, but it will require everyone who is going to view the data to have either an E5 SKU or a Power BI license. We aren't going to cover that in this walk through, I'll cover that in more detail in another post.

You'll want to click the Sign In button on Power BI, if you haven't used it before, it should be a simple sign up process. When you log in, you'll land in your workspace. To get started, click the big **Get Data** button in the lower left corner. If you dont see it, click the hamburger meun in the top left corner of the screen.

![Power BI Get Data](http://ericjalexander.com/img/powerbi_getdata.png "Get Data")

In the Content Pack section, click the **Get** button for Services.

![Power BI Services](http://ericjalexander.com/img/powerbi_services.png "Services")

Find Google Analytics in the list and click the **Get it now** button. You'll get a popup asking you to connect to Google Analytics. Click the Sign in button and Allow it to have off line access.

![Power BI OAuth](http://ericjalexander.com/img/powerbi_oauth.png "OAuth")

Configure the dialog to point to the analytics information you want to show and click Import, and that is it!

![Power BI Import data](http://ericjalexander.com/img/powerbi_import.png "Import data")

The only thing left to do is to click the Share icon in the upper right of the screen to let other users see the juicy data while you look like a rockstar.

![Power BI victory](http://ericjalexander.com/img/powerbi_victory.png "Winning")

## Final Thoughts

With a few pretty simple steps, we've completely transformed the way we see our SharePoint sites being used. In a follow up article, I'll show how to make a slight tweak to the tracking code in analytics to capture user information so you can really get a sense of who is doing what.

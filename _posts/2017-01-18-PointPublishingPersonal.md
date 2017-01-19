---

layout: post

title: "PointPublishingPersonal#0, What the heck is it?"

subtitle: "Some foreign site just showed up"

date: 2017-01-18

category: Administration

tags: Office 365, Delve, PointPublishingPersonal#0, PowerShell

author: Eric

---

Welcome back friends. Today's post is going to cover a strange anamoly I hadn't come across before, and I couldn't find much about it, so I thought I'd be a swell guy and share it.

# Backstory
In the latest release of the [SharePoint Online Management Shell](http://www.microsoft.com/en-us/download/details.aspx?id=35588SharePoint Online Management Shell) (16.0.6008.1200), we got some much needed functionality that now returns Office 365 Groups and Video Channels. Previously these were buried away and you needed a cumbersome REST query to get to the video channels or dial into Exchange Online and find unified groups for groups. This is now all attainable via `Get-SPOSite -Limit All -Template "Group#0"` or `Get-SPOSite -Limit All -Template "PointPublishingTopic#0`.

What I wasn't anticipating when running a -Limit all was a handful of sites with a PointPublishingPersonal#0 template with a URL like `https://tenant.sharepoint.com//portals/personal/ericalexander`. I binged around ad all I could really find was articles that scraped [TechNet](https://blogs.technet.microsoft.com/wbaer/2015/09/07/sharepoint-server-2016-it-preview-web-templates/) saying it was:

 > A site template used to create a pointpublishing personal site.
 
 Ya, real helpful. So I took my own advice from my [previous post](http://ericjalexander.com/blog/2016/12/27/Transitioning-to-SPO-Admin) and got a hold of Microsoft support and this is what I found, I have't tried to confirm this with Premier support or any of my insiders yet.
 
### The Who

Any licensed user can do this. I was perplexed as to where these came from as we have a restricted provisioning process in place. So I know it didn't come through our normal process flow.

### The What
 
 The PointPublishingPersonal#0 template is a hidden site collection with an **unconfigurable default site quota of 1 TB** that counts against your tenant storage. If deleted, this site and it's contents cannot be recovered. The only way to restrict this functionality is to turn off Delve.
 
### The Where
 
 You may be wondering where it originates from. The answer is Delve. If you go to Delve and click on Me in the left navigation, you'll be taken to your bio. At the very bottom, there is a section called Blog.
 
 ![Blog Section](http://ericjalexander.com/img/delve_me.png "Blog section") 
 
### The How
 
 By clicking on the + New Post link, it will generate a new blog site, aka PointPublishingPersonal#0. Then you'll end up on a page like this:
 ![Modern blogging in Delve](http://ericjalexander.com/img/delve_blog.png "Modern blogging in Delve") 
 
### The Why
 
 This is pure speculation, but I imagine that they are trying to kill off the standard SharePoint functionality in OneDrive for Business. In SharePoint versions past, it was very common to have a blog subsite hanging off your MySite. The move to Delve makes sense as this is where your profile data is just like MySites of old and gives the modern publishing experience like we have in team sites and Office Groups.

### Where do we go from here?

I tried getting a handle on one of thee sites to see if I could set a quota, I couldn't. I thought surely I could enforce some sort of policy from the compliance center, I could not. This puts us in a tough spot. If these sites truely count towards tenant storage, then 1TB per person could cripple you entirely. Our group will continue to monitor the usage of these sites, if we see a spike in storage from them it will most likely lead to  making sure their content is stored in OneDrive for Business or a team/group.

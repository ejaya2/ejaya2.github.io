---

layout: post

title: "Transitioning to a SharePoint Online Administrator"

subtitle: "Forget everything you thought you knew, it doesn't matter anymore"

date: 2016-12-27

category: Administration

tags: Tips, Discussion

author: Eric

---

Welcome back friends. Today in this catchy title, I wanted to go over some of the things that might trip you up as you transition to SharePoint Online. This caters specifically to SharePoint Online, if you are running hybrid, then you still need to remember all your on premises information.

## It's a Service, and it is not ours

In most organizations, the IT department is a service center, offering tools and services for the organization to operate. These are systems like line of business systems to financial systems and HR systems and maintaining and upgrading workstations, AD, security, and so many other functions. This can also mean collaboration systems like SharePoint. We would manage the infrastructure and have administrators and developers in place to make sure the system is operating optimally and provide value add functionality for the business users.

In the SharePoint Online space, the infrastructure and farm are abstracted away from us and is managed by Microsoft. It is their responsibility to make sure the health of the service is running optimally. We no longer have tools like correlation logs at our fingertips to diagnose and fix issues.

## We are the middleman

In the Office 365 paradigm, IT is now the middleman in this service offering. We don't have the resources available to fix what we would consider normal problems. We have to gather information from the user, replicate it, and defer to Microsoft. This can be in one of a few ways:

 * Sites like [SharePoint.StackExchange](http://sharepoint.stackexchange.com) or the [MSDN forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=sharepointgeneral)
 * Office 365 Support requests
 * Microsoft Premier Support (if your organization has this available)  
 
Some people might balk at the idea of using Premier Support, however, it is well worth it if you do. In a few Enterprise Agreements I've seen, Office 365 Premier Support was included. Another benefit of using Premier is that your TAM is in the loop on these issues. Often you can receive rebates as part of your EA based on the service unavailability. This documents and brings those issues to light. 
 
Another tangible example, from personal experience, is that there are complex problems the Office 365 Support folks just won't be able to solve. Going directly to Premier can save time. As part of a site collection we were delivering, there was a requirement to allow users to create search based alerts. You can set these up in the UI, but you'll notice you'll never receive an alert when new content is added. I went directly to Premier. They reproduced the issue, brought it to the product group's attention, they found the problem, and currently are working on a fix. Cutting out the initial Office 365 Support got this moving along quicker.
 
## In Conclusion
 
The transition from an on-prem administrator to a SharePoint Online administrator can be challenging at times. Most of the PowerShell you've learned along the way will still be relevant, there are just some new restrictions and ways we have to work with the environment. But the biggest take away is don't be afraid to ask for help. We just don't have the access like we used to and the sooner you can come to grips with this, the easier your administration duties will become.

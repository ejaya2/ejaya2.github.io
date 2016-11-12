---

layout: post

title: "Setting Up PowerShell for Office 365, part 2"

subtitle: "Getting the plumbing in order for successfully managing your tenant"

date: 2016-XX-XX

category: Administration

tags: PowerShell

author: Eric

---

Welcome back friends. In part 2 of Setting up PowerShell for Office 365, I'm going to show you how to create a profile, wire up all of our previously installed modules necessary to talk to SharePoint, and show a technique on sharing a module across a team. If you don't know what I'm talking about, then please refer to the [previous article](http://ericjalexander.com/blog/2016/11/03/Setting-Up-PowerShell).



## Creating A Profile

What is a PowerShell profile and why is it useful? A profile is a ay to load scripts and modules every time a PowerShell session is launched. This is beneficial in that as soon as the ISE or console gives you the command prompt, you are ready to start working. In a team setting, it enforces a consistent experience from team member to team member. On a personal stand point, I want the ISE to be ready with everything I need when it is available and I don't want to have to always bloat my code with dependency type injection to load modules that might not have been loaded.

Creating a profile is pretty simple. From the Start menu, launch Windows PowerShell or Windows PowerShell ISE. Profiles are specific to which application you decide to use. If you use both the console and ISE equally, you'll need to repeat this process for each application. To see if you have a profile, you would type this into the console:

```PowerShell
Test-Path $profile
```
![Test-Path Syntax](http://ericjalexander.com/img/testprofile.png "Test-Path syntax")

It returns a boolean, with **False** being no profile exists or **True** meaning a profile already exists. In the case of **False**, then you'll want to run the following to generate your profile:

```PowerShell
New-Item -path $profile -itemtype file -force
notepad $profile
```
This will create a file named `Microsoft.PowerShellISE_profile.ps1` or `Microsoft.PowerShell_profile.ps1` in the `C:\Users\<username>\Documents\WindowsPowerShell` directory, and open NotePad to edit it.

![New Profile Syntax](http://ericjalexander.com/img/newprofile.png "New profile syntax")

If the `Test-Path $profile` returned **True**, you can simply forgo `New-Item` command and simply do `notepad $profile` to open your existing profile.

Now that we have our profile created and it open for editing, it is time to wire in all our modules so PowerShell is ready for us when we launch it.

## Wiring Components

The first thing we want to do is import the SharePoint Online management shell. This is done by utilizing the [Import-Module](https://technet.microsoft.com/en-us/library/hh849725.aspx) command in our profile:
```PowerShell
#Bring in the Sharepoint modules
Import-Module "C:\Program Files\SharePoint Online Management Shell\Microsoft.Online.SharePoint.PowerShell" -DisableNameChecking
```
With the Management Shell imported, the next required components we want are the client components so we can utilize the client side object model (CSOM). In my profile, I've decided to load most of the common ones we would need. The first two below are absolutely required, the remaing 4 are for convenience so I don't have to remember to load them on demand. 

```PowerShell
#Add references to SharePoint client assemblies - required for CSOM
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Runtime.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.UserProfiles.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.Office.Client.Policy.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Publishing.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Taxonomy.dll"
```
The next module is optional and I explain them in more detail in the next section. This imports a shared module comprised of custom functions for our team to do consistent work.

```PowerShell
#My extensions
Import-Module "C:\Users\<your UID>\SharePoint\Sharepoint - Scripts\Powershell\OI.PowerShell.Extensions.psm1"
```

The last section contains some global variables and the connection scheme for initiating the connection to SharePoint Online and persisting our credentials while PowerShell is open. This becomes increasingly handy as we can instantiate CSOM contexts without constantly having to ask for credentials.

```PowerShell
#Global Variables
$CAprod = "https://tenant-admin.sharepoint.com"
$MysiteHost = "https://tenant-my.sharepoint.com"
$me = "firstname.lastname@domain.com"
$credential = Get-Credential -Message "Please enter your Office 365 Administrative credentials" -UserName $me


Connect-SPOService -Url $CAprod -Credential $credential
Write-Host -ForegroundColor Green "All modules loaded and connected to TenantAdmin, let's get started!"
```

Putting it all together, you'll have a profile that will look like this:

```Powershell
#Bring in the Sharepoint modules
Import-Module "C:\Program Files\SharePoint Online Management Shell\Microsoft.Online.SharePoint.PowerShell" -DisableNameChecking

#Add references to SharePoint client assemblies - required for CSOM
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Runtime.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.UserProfiles.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.Office.Client.Policy.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Publishing.dll"
Import-Module "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Taxonomy.dll"

#My extensions
Import-Module "C:\Users\<your UID>\SharePoint\Sharepoint - Scripts\Powershell\OI.PowerShell.Extensions.psm1"

#Global Variables
$CAprod = "https://tenant-admin.sharepoint.com"
$MysiteHost = "https://tenant-my.sharepoint.com"
$me = "firstname.lastname@domain.com"
$credential = Get-Credential -Message "Please enter your Office 365 Administrative credentials" -UserName $me

Connect-SPOService -Url $CAprod -Credential $credential
Write-Host -ForegroundColor Green "All modules loaded and connected to TenantAdmin, let's get started!"
```

Save and close notepad and fire up PowerShell again. If all worked correctly, you should be challenged with a credential window, prepopulated with your email address asking you for your password. Supply it and you are connected to SharePoint Online!

![Credential prompt](http://ericjalexander.com/img/credential.png "Credential prompt")

## Working Collaboratively

One of the big pitfalls with administrating SharePoiint Online with PowerShell is the limited number of commandlets. When I first started using the management shell, there were about 12 commands available. In subsequent releases, there are more commands but still nowhere near the same as an on premises installation. For example, you might want to use `Get-SPWeb` or `Get-SPList` for a lot of heavy lifting, there is no equivalent in the SharePoint Online Management Shell. Or perhaps you were used to navigating through your web applications by using `Get-SPWebApplication`, In SharePoint Online, we do not have access to that level of the farm. This means we have to use open source solutions like the [Office 365 Dev PnP PowerShell CmdLets](https://github.com/OfficeDev/PnP-PowerShell) mentioned in my last post, or building out your own module with your own business logic. I was in the later camp, so most of my subsequent posts will operate under that assumption. The module to date is over 4000 lines of code.

Having said all of that, the module we have is stored in SharePoint Online under version control. This allows us to use OneDrive for Business to sync the library to our local machines and pull in our module. You can even take it a step further if you'd like by requiring check out and content approval to tighten the authorship of adding new functions into the module. Other organizations might do things differently, we've found this works well for all the SharePoint administrators on the team, and the folks who manage Exchange Online and Skype for Business are doing similar things. Also as shown above, I wired in my own module stored in my OneDrive for Business site to include some functions I use from time to time that aren't a necessity for inclusion into the main module. These could be your own utility functions for sending custom emails or particular reports that someone asks you for infrequently.

## Conclusion
In this article, I've shown you how to generate a profile, wire in all the necessary modules needed to work with SharePoint Online, and provided a interesting approach to share custom scripts and modules with team members. With all of this in place, you are ready to start your SharePoint Online management ~~nightmare~~ journey. Look for future articles about creating a shared module of functions for your team and other things PowerShell related.
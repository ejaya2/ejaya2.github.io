---

layout: post

title: "Generate a Full Permission Report in PowerShell"

subtitle: "See who has access to what in your SharePoint Online sites via PowerShell"

date: 2016-11-20

category: Administration

tags: PowerShell, Permissions

author: Eric

---

Welcome back friends. Today I'll be going over how to create a script that will iterate over every site in your tenancy and emit a permissions report down to the list level on who has access to what information.

In a [previous post](http://ericjalexander.com/blog/2016/11/12/Setting-Up-PowerShell-pt2), I lamented how few cmdlets there are available to SharePoint Online administrators versus local installations. For those running SharePoint locally, in about 6 lines of PowerShell you can iterate through your entire farm and output all the permissions. There are even third party ISVs that help with these tasks, the problem I was running into was these tools would take a long time to run and ultimately not get me the information I was looking for. So how do we adapt something like this to SharePoint Online?

## Remarks

The script I came up with, primarily for our needs, iterates through all of the Team and Project spaces and will detail permissions. Things to note:

 * It assumes you have a profile in place like I [previously described](http://ericjalexander.com/blog/2016/11/12/Setting-Up-PowerShell-pt2)
 * It has a dependency on [Gary LaPointe's](https://www.itunity.com/users/gary-lapointe) immensely [Load-CSOMProperties](https://www.itunity.com/article/loading-specific-values-lambda-expressions-sharepoint-csom-api-windows-powershell-1249) function. I have this included into our module, so it is loaded with our profiles. If you don't have it, you will need to include the function in this script.
 * It only details things with unique permissions, lists or subsites with inherited permissions are not included
 * It will output if any of the broad claims are being used. In our tenant, we decided to hide these claims in a [Set-SPOTenant](https://technet.microsoft.com/en-us/library/fp161390.aspx "Set-SPOTenant") command, but they can still be used.
 * It will enumerate group membership, so the output can be long
 * It will output the display names of users and AD groups that are used for permissions
 * Update the configuration section as desired to match your tenant

This script was adapted from the PnP group's [CSOM example](https://github.com/OfficeDev/PnP/blob/master/Samples/Core.PermissionListing/Core.PermissionListingWeb/Pages/Default.aspx.cs) of enumerating permissions. The challenge in their script was overcoming the lambda in their script which PowerShell has no equivalent. This is where Gary's utility function comes into play and is the critical piece to allow us to do this in PowerShell. 

I've tried to expalin what is going on in the script with comments and some rationale, based on our needs.

## Running a permissions report

Running the script is easy, simply make the desired configuration changes in the configuration section and run it from your ISE. It can be a set up to be a scheduled report if you so choose. 

## The script

You can download the script from [here](http://ericjalexander.com/scripts/enumerate_permissions.ps1)

```PowerShell
#Adapted from: https://github.com/OfficeDev/PnP/blob/master/Samples/Core.PermissionListing/Core.PermissionListingWeb/Pages/Default.aspx.cs
#Help from gary LaPointe: https://www.itunity.com/article/loading-specific-values-lambda-expressions-sharepoint-csom-api-windows-powershell-1249 
#Only logs things with unique permissions, subsites and list which inherit from the parent are not included in the output


#region configuration
#the filename and path to store the file
$filename = "C:\Users\$env:USERNAME\Desktop\PermissionsReport.csv"
$domainSuffix = "*domain.com"

#email configuration options
$To = "someone@domain.com"
$From = "no-reply@sharepointonline.com"
$Subject = "SharePoint Online Permissions Report"
$Body = "Attached is the output of the permissions report."
$SMTPServer = "smtp.domian.com"
#endregion

#region functions
function Process-RoleAssignments($securableObject, $clientContext, $SiteUrl){
    $objResults = @()
    #This line was from the PnP implementation, since there are no lambdas in PowerShell
    #we use Gary's Get-CSOMProperties function to get it
    #$clientContext.Load($securableObject, $x => $x.HasUniqueRoleAssignments)
    Load-CSOMProperties -object $securableObject -propertyNames @("HasUniqueRoleAssignments")
    $clientContext.ExecuteQuery()

    #if the object has unique permissions we will process it, if it inherits from the parent we skip it
    if($securableObject.HasUniqueRoleAssignments){
        $roleAssignments = $securableObject.RoleAssignments
        $clientContext.Load($roleAssignments)
        $clientContext.ExecuteQuery()

        foreach ($roleAssignment in $roleAssignments){
            $member = $roleAssignment.Member
            $roleDef = $roleAssignment.RoleDefinitionBindings

            $clientContext.Load($member)
            $clientContext.Load($roleDef)
            $clientContext.ExecuteQuery()

            foreach ($binding in $roleDef){
                #We are skipping role bindings of limited access, they should get picked up at another point 
                if($binding.Name -ne "Limited Access" ){
                    #write-host "$($member.PrincipalType) $($member.LoginName) $($binding.Name)" -ForegroundColor White
                    #if the principal type is a SharePointGroup
                    if($member.PrincipalType -eq "SharePointGroup"){
                        #Get the group membership
                        $group = Get-SPOSiteGroup -Site $SiteUrl -Group $member.LoginName
                        #if the group has users in it
                        if($group.Users.Count -gt 0){
                            #run the group members through the group processing function to get the display names
                            $groupMembership = Process-GroupMembers -SiteUrl $SiteUrl -members $group.Users
                        }
                        else{
                            #an empty group has permission to the object, need to log it for transparency
                            $groupMembership = "Empty Group"
                        }
                        $objResults += New-Object PSObject -Property @{
                            "Principal" = $member.LoginName
                            "Role" = $binding.Name 
                            "Members" = $groupMembership
                            "Everyone" = $group.Users.Contains("true")
                            "EveryoneExcept" = $group.Users.Contains("spo-grid-all-users")
                            "NTAuthority" = $group.Users.Contains("windows")
                        }                        
                    }
                    #otherwise it is a user
                    else {
                        #run the user through the group processing function to get the display name
                        $userName = Process-GroupMembers -SiteUrl $SiteUrl -members $member.LoginName.Split("|")[2] 
                        $objResults += New-Object PSObject -Property @{
                            "Principal" = "Explicit User"  
                            "Role" = $binding.Name
                            "Members" = $userName
                            "Everyone" = $($member.LoginName.Split("|")[1] -eq "true")
                            "EveryoneExcept" = $($member.LoginName.Split("|")[2] -like "spo-grid-all-users*")
                            "NTAuthority" = $($member.LoginName.Split("|")[1] -eq "windows")
                        }
                    }
                }
            }
        }
    }

    return $objResults
}
function Process-GroupMembers($SiteUrl, $members){
    #The output of the get-spogroup gives the users login name
    #this function processes the results and returns the membership as display names of people and groups
    $returnMembers = @()
    foreach($member in $members){
        #Everyone claim
        if($member -eq "true"){
            $returnMembers += "Everyone"
        }
        #Everyone except external users claim
        elseif($member -like "spo-grid-all-users*"){
            $returnMembers += "Everyone except external users"
        }
        #Sharepoint account
        elseif($member -eq "SHAREPOINT\system"){
            $returnMembers += $member
        }
        #a named user
        elseif($member -like $domainSuffix){
            $user = get-aduser -Filter {mail -eq $member}
            $returnMembers += $user.Name
        }
        #an AD group
        elseif($member -like "s-1*"){
            $group = get-spouser -Site $SiteUrl -LoginName "c:0-.f|rolemanager|$member"
            $returnMembers += $group.DisplayName
        }
        #windows claim, probaly only seen if migrated to SPO from on-prem
        elseif($member -eq "windows"){
            $returnMembers += "NT AUTHORITY\authenticated users"
        }
        #if it didn't match above, IDK what it is!
        else{
            $returnMembers += $member
        }
    }
    #return the results
    return $returnMembers
}
#endregion

#capture output
$Results = @()
#get sites
$sites = get-sposite -detailed -limit All

#process sites
foreach($site in $sites){
    #only looking at project and team sites
    if($site.Template -eq "PROJECTSITE#0" -or $site.Template -eq "STS#0"){
        write-host "Processing site collection - $($site.Title)"
        $ctx = New-Object Microsoft.SharePoint.Client.ClientContext($site.Url)
        $ctx.Credentials = New-Object Microsoft.SharePoint.Client.SharePointOnlineCredentials($credential.UserName,$credential.Password)

        #root site
        $web = $ctx.Web
        $ctx.Load($web)
        #Get the web permissions
        $rootResults = Process-RoleAssignments -securableObject $web -clientContext $ctx -SiteUrl $site.Url
        foreach($webresult in $rootResults){
            #Build a new object from the results
            $results += New-Object PSObject -Property @{
                "Site Name" = $site.Title
                "URL" = $site.Url 
                "SecurableObject" = $web.Title
                "Principal" = $webresult.Principal 
                "Role" = $webresult.Role
                "Members" = $webresult.Members
                "Everyone" = $webresult.Everyone
                "EveryoneExcept" = $webresult.EveryoneExcept
                "NTAuthority" = $webresult.NTAuthority
            }
        }
    
        #root site Lists
        $lists = $web.Lists
        $ctx.Load($lists)
        $ctx.ExecuteQuery()
        foreach ($list in $lists){
            #skipping over hidden lists
            if(!$list.Hidden){
                $ctx.Load($list.RootFolder)
                $ctx.ExecuteQuery()
            
                $rootListResults = Process-RoleAssignments -securableObject $list -clientContext $ctx -SiteUrl $site.Url
                foreach($listresult in $rootListResults){
                    $results += New-Object PSObject -Property @{
                        "Site Name" = $site.Title
                        "URL" = $list.RootFolder.ServerRelativeUrl
                        "SecurableObject" = "List - $($list.Title)"
                        "Principal" = $listresult.Principal 
                        "Role" = $listresult.Role
                        "Members" = $listresult.Members
                        "Everyone" = $listresult.Everyone
                        "EveryoneExcept" = $listresult.EveryoneExcept
                        "NTAuthority" = $listresult.NTAuthority
                    }
                }
            }
        }
    
        if($site.WebsCount -gt 0){
            #subsites
            $webs = $web.Webs
            $ctx.Load($webs)
            $ctx.ExecuteQuery()

            foreach ($subWeb in $webs){
                $subWebResults = Process-RoleAssignments -securableObject $subWeb -clientContext $ctx -SiteUrl $site.Url
                foreach($subwebresult in $subWebResults){
                    $results += New-Object PSObject -Property @{
                        "Site Name" = $site.Title
                        "URL" = $subWeb.Url
                        "SecurableObject" = "Subsite - $($subWeb.Title)"
                        "Principal" = $subwebresult.Principal 
                        "Role" = $subwebresult.Role
                        "Members" = $subwebresult.Members
                        "Everyone" = $subwebresult.Everyone
                        "EveryoneExcept" = $subwebresult.EveryoneExcept
                        "NTAuthority" = $subwebresult.NTAuthority
                    }
                }
                #subsite lists
                $subLists = $subWeb.Lists
                $ctx.Load($subLists)
                $ctx.ExecuteQuery()
                foreach ($subList in $subLists){
                    #skipping over hidden lists
                    if(!$subList.Hidden){
                        $ctx.Load($subList.RootFolder)
                        $ctx.ExecuteQuery()

                        $subWebListResults = Process-RoleAssignments -securableObject $subList -clientContext $ctx -SiteUrl $site.Url
                        foreach($subweblistresult in $subWebListResults){
                            $results += New-Object PSObject -Property @{
                                "Site Name" = $site.Title
                                "URL" = $subList.RootFolder.ServerRelativeUrl
                                "SecurableObject" = "List - $($subList.Title)"
                                "Principal" = $subweblistresult.Principal 
                                "Role" = $subweblistresult.Role
                                "Members" = $subweblistresult.Members
                                "Everyone" = $subweblistresult.Everyone
                                "EveryoneExcept" = $subweblistresult.EveryoneExcept
                                "NTAuthority" = $subweblistresult.NTAuthority
                            }
                        }
                    }
                }
            }
        }
    }
}

#Pipe the results out to a csv
$Results | Select "Site Name","URL","SecurableObject","Principal", "Role",@{n="Members";e={(@($_.Members) | Out-String).Trim()}},"Everyone", "EveryoneExcept", "NTAuthority" | Export-Csv $filename -NoTypeInformation
Write-Host "Report saved to $filename" -ForegroundColor Green

#Send email 
Send-MailMessage -Attachments $filename -Body $Body -From $From -To $To -Subject $Subject -BodyAsHtml -SmtpServer $SMTPServer

``` 


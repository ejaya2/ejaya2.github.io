---

layout: post

title: "Deleting Office Groups will Leave Orphans in SharePoint Online"

subtitle: "Why are these features half baked"

date: 2017-02-13

category: Administration

tags: [PowerShell,Office365]

author: Eric

---

Welcome back friends. Just a little venting in today's post with a code snippet to help your administration duties.

Office Groups are the direction Microsoft is pushing for Office 365 users. In my opinion, the tradition site collections of SharePoint are going to go the way of the dodo. Groups offer so much more flexibility and capability than a traditional SharePoint site with things like Power BI, Planner, and Teams. What really annoys me is the way these features get realeased with little or no administrative options in the respective UIs, and partially functioning PowerShell APIs. What I would **love** to see is a feature not ship until it has full administrative CRUD operations in PowerSHell at the very least. In order for it to be ready, it must be creatable, readable, updatable, and deletable to be ready.

This brings my to my latest vexation with Office Groups. We had a slew of dead and test Office Groups to clean up. These were deleted in the Office 365 Admin Center, a pretty straight forward deletion of the group from the Groups section. When deleting the group, you get a warning that it can be up to 24 hours until the identifier is available for reuse. Cool, I assumed that there was a wave of backend jobs that process at later times cleaning all the components up. Well, you know what they say about assumptions.

The next day, I ran my trusty `Get-SPoSite -Template "GROUP#0" -Limit All` commandlet and low and behold, the sites still exist even though there are no traces of the group any longer. Now what? Start venting on twitter obviously. I had a nice conversation with [Brian T. Jackett](https://twitter.com/BrianTJackett) about this. I already had a semblence of what I wanted to do, but he was kind and gave me a kick start on the PowerShell. I took that, added to it and generalized it as a function. It is:

```PowerShell
Function Delete-OrphanGroupSites{
    <#
        .Synopsis
           Deletes orphaned SharePoint Group sites
        .DESCRIPTION
           Deletes the SharePoint sites of Office groups that no longer exist
        .EXAMPLE
           Delete-OrphanGroupSites -Confirm
    #>
    Param
    (
        [Parameter(Mandatory=$false,HelpMessage="If included, it will delete the orphaned sites",Position=0)]
        [switch]$Confirm
    )
    begin{
        $exoSession = ""
        $sessions = Get-PSSession
        foreach($session in $sessions){
            if($session.ComputerName -eq "outlook.office365.com" -and $session.State -eq "Opened"){
                $exoSession = $session
                break
            }   
        }
        if(!$exoSession){
            Connect-ExchangeOnline
        }
    }
    process{
        $UnifiedGroupSiteURLs = Get-UnifiedGroup | select -Property SharePointSiteUrl 
        $SPOGroupSiteURLs = Get-SPOSite -Template GROUP#0 -Limit All | select -Property Url | Sort-Object Url

        $sitesToPurge = $SPOGroupSiteURLs | where URL -NotIn $UnifiedGroupSiteURLs.SharePointSiteUrl
        write-host "==================================================================" -ForegroundColor Magenta
        write-host "Orphaned SPO Group sites:" $sitesToPurge.Count -ForegroundColor Magenta
        write-host "These are sites that exist in SPO but no longer in Exchange" -ForegroundColor Magenta
        write-host "==================================================================" -ForegroundColor Magenta
        $sitesToPurge

        if($Confirm){
            foreach($site in $sitesToPurge){
                try{
                    Remove-SPOSite -Identity $site.Url -confirm
                    Write-Host "Orphaned site removed: " $site.Url -ForegroundColor Green
                }
                catch{
                    write-host  $_.Exception.Message
                }
            }    
        }
        else{
            write-host "In order to delete these sites, you must use the switch parameter -Confirm, Delete-OrphanGroupSites -Confirm" -ForegroundColor Yellow
        }
    }
    end{}
}
```

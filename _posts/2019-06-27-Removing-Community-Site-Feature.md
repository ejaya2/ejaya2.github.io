---
layout: post
title: "Removing Community Site Feature Components"
subtitle: "Clean up this garbage feature"
date: 2019-06-27
category: Workflow
tags: [PowerShell,2013,Features]
author: Eric
---

In today's post, I wanted to go over how to clean up the components of the Community Site Feature. This is the one that will let users turn a Team Site into a site that functions like a community site.

# The Set Up

One of our users was trying to set up some things in their site and had stumbled on this feature. Not knowing what it did, they enabled it, tested some stuff out, decided they didn't like it, couldn't remove it, and called us. One of our analysts couldn't understand why the lists couldn't be deleted so she got me involved. This feature:
* Creates 6 specific pages in the Site Pages library
* Provisions 5 lists (Badges, Categories, Community Members, Discussions List, Reports List)
* Provisions web parts

# The Problem

Several of the lists are interconnected with lookup columns and have single and composite indexes. This made it challenging to remove. [This SharePoint.StackExchange post](https://sharepoint.stackexchange.com/questions/97342/how-to-delete-these-lists-linked-to-other-lists-generated-by-community-feature) got me started. It only cleaned up 2 of the lists so I started digging around further.

Complications arose when trying to delete lookup columns that were part of composite indices or dependent lookups for other columns. With the help of SPManager, I was able to finally track all these down. I finally came up with a script that, to the best of my ability and knowledge, cleans everything up.

Another problem I saw thrown around, but not substantiated on my end, was that by enabling these features, it doesn't make the site visible in the central Community Portal.

# The Fix
In the below script, simply replace the URL in line 1 and it should clean out the garbage this feature creates.

```
$web = "https://sp.domain.com/sites/erictest126"

# The Web
$spweb = Get-SPWeb $web

$communityFeatureVisibleInUI = "961d6a9c-4388-4cf2-9733-38ee8c89afd4" 
$communityFeatureHiddenInUI = "4326e7fc-f35a-4b0f-927c-36264b0a4cf0"
# Disable the Community Site feature
if(Get-SPFeature -Identity $communityFeatureVisibleInUI -Web $spweb.Rootweb.Url -ErrorAction SilentlyContinue){
    Write-Host "Deactivating Community Site Feature on" $web
	Disable-SPFeature -Identity $communityFeatureVisibleInUI -Url $web -Confirm:$False
}

# The lists
$badgeList = $spweb.Lists["Badges"]
$categoriesList = $spweb.Lists["Categories"]
$membersList = $spweb.Lists["Community Members"]
$discussionsList = $spweb.Lists["Discussions List"]
$abuseList = $spweb.Lists["Reports List"]
$sitePages = $spWeb.Lists["Site Pages"]

# Holding arrays
$communityPages = @()
$communityMembers = @()
$discussionsFields = @()
$membersIndexes = @()
$discussionsIndexes = @()

# Disable Content Types on lists
$membersList.ContentTypesEnabled = $false
$membersList.Update()
$discussionsList.ContentTypesEnabled = $false
$discussionsList.Update()
$abuseList.ContentTypesEnabled = $false
$abuseList.Update()

# Find the feature generated pages
foreach($page in $sitePages.Items){
    if($page.Name -eq "Community Home.aspx" -or $page.Name -eq "About.aspx" -or $page.Name -eq "Topic.aspx" -or $page.Name -eq "Category.aspx" -or $page.Name -eq "Categories.aspx" -or $page.Name -eq "Members.aspx"){
        # Add it to the communityPages array, the item cannot be deleted from the collection proper
        $communityPages += $page
    }
}

# Delete the community pages
foreach($communityPage in $communityPages){
    $communityPage.Recycle()
}

# Find any created Community Members
foreach($cm in $membersList.Items){
    # Add it to the communityPages array, the item cannot be deleted from the collection proper
    $communityMembers += $cm
}

# Delete the created members
foreach($member in $communityMembers){
    $member.Recycle()
}

# Remove the hidden Badge list, this is the easiest to remove
$badgeList.AllowDeletion = $true
$badgeList.Update()
$badgeList.Delete()

# Remove the Categories List
$categoriesList.AllowDeletion = $true
$categoriesList.Update()
$categoriesList.Delete()

# Remove the Reports List
$abuseList.AllowDeletion = $true
$abuseList.Update()
$abuseList.Delete()

# Column fixup
$BadgeColumn = $membersList.Fields.GetFieldByInternalName("GiftedBadgeLookup")
$BadgeColumn.AllowDeletion = $true
$BadgeColumn.Sealed = $false
$BadgeColumn.Delete()

$discussionDependantColumns = @("AbuseReportsCommentsLookup","AbuseReportsReporterLookup","AuthorGiftedBadgeLookup","AuthorLastActivityLookup","AuthorMemberSinceLookup","AuthorMemberStatusIntLookup","AuthorNumOfBestResponsesLookup","AuthorNumOfPostsLookup","AuthorNumOfRepliesLookup","AuthorReputationLookup")
foreach($discussionDependantColumn in $discussionDependantColumns){
    $LookupColumn = $discussionsList.Fields.GetFieldByInternalName($discussionDependantColumn)
    $LookupColumn.AllowDeletion = $true
    $LookupColumn.Sealed = $false
    $LookupColumn.Delete() 
}

# Now the lookup column can be deleted
$ReportColumn = $discussionsList.Fields.GetFieldByInternalName("AbuseReportsLookup")
$ReportColumn.AllowDeletion = $true
$ReportColumn.Sealed = $false
$ReportColumn.Delete()

$MemberLookupColumn = $discussionsList.Fields.GetFieldByInternalName("MemberLookup")
$MemberLookupColumn.AllowDeletion = $true
$MemberLookupColumn.Sealed = $false
$MemberLookupColumn.Delete()


# Deletes the Members column
$MemberColumn = $membersList.Fields.GetFieldByInternalName("Member")
$MemberColumn.AllowDeletion = $true
$MemberColumn.Sealed = $false
$memberColumn.EnforceUniqueValues = $false
$memberColumn.Indexed = $false
$MemberColumn.Delete()

# Delete community members list indicies
foreach($listIndex in $membersList.FieldIndexes){
    $membersIndexes += $listIndex.Id
}

foreach($index in $membersIndexes){
    $membersList.FieldIndexes.Delete($index)
}

$membersList.AllowDeletion = $true
$membersList.Update()
$membersList.Delete()


foreach($listIndex in $discussionsList.FieldIndexes){
    $discussionsIndexes += $listIndex.Id
}

foreach($index in $discussionsIndexes){
    $discussionsList.FieldIndexes.Delete($index)
}

# Discussions List column fixup
$Categories = $discussionsList.Fields.GetFieldByInternalName("CategoriesLookup")
$Categories.Indexed = $false
$Categories.AllowDeletion = $true
$Categories.Sealed = $false
$Categories.Delete()

# Finally delete the Discussions List
$discussionsList.AllowDeletion = $true
$discussionsList.Update()
$discussionsList.Delete()

# Disable the hidden Community Site feature, cleans up content type
if(Get-SPFeature -Identity $communityFeatureHiddenInUI -Site $web -ErrorAction SilentlyContinue){
    Write-Host "Deactivating Community Site hidden feature on" $web
	Disable-SPFeature -Identity $communityFeatureHiddenInUI -Url $web -Confirm:$False
}
```

# Conclusion
If users want community functionality, create sites based on the Community Site template, don't hack it in with this feature.

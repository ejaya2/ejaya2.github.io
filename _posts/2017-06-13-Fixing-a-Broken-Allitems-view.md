---

layout: post

title: "Fixing a Broken AllItems View"

date: 2017-06-12

category: Administration

tags: [PowerShell,WebPart,Document Library]

author: Eric

---


Today I want to detail an issue I've encountered that is a bit tricky, repairing the default view after the web part has been deleted.

# The Set Up

A user was trying to add some web parts into the All Items view of a document library. Upon not getting the results they were after, the web parts were removed. However, even the list view was removed too.

This caused panic as they thought the items were gone too, which we know as administrators, is not the case. So to initially try and fix the problem, the page was edited and the app part was added back into
the web part zone. However this did not have the desired view, it went back to an old style view. I had to find a fix.

# The Constraints 

In the past, I recall fixing this easily enough via Designer. However, Designer in our environment is disabled from the farm level. This means even administrators can't use it. I had to think of another way.

# The Fix

The first thing I did was through the UI, created a new view using the Standard View view type. I added the desired fields and sort order and saved the view. I also made the mistake of setting it as the default view. Don't do that.

Next, I went to the AllItems.aspx page and put the page into edit mode. I then added the list app part into the page and configured the web part to use the newly created view. This rendered properly but now I had to fix my error of this no longer being the default view. With my constraint above, I had to turn to PowerShell.

I ran the below to find all the views in my document library.
```PowerShell
$web = get-spweb -Identity https://intranent.contoso.com/sites/theSite
$list = $web.Lists | Where-Object {$_.Title -eq "The Document Library in Question"}
$list.Views
```
The output showed me that the name of the view which had a DefaultViewUrl of allitems.aspx didn't have a Title. So I added the below 3 lines and reran the script.
```PowerShell
$view = $list.Views[""]
$view.DefaultView = $true
$view.Update()
```
This updated the default view of the list to the newly repaired view. 

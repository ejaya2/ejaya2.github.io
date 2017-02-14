---
layout: post
title: "Error when clicking on a list item with many workflows"
subtitle: The craziest things you'll run into sometimes
date: 2012-05-24
category: "Power User"
tags: [Workflow]
author: Eric
---

In our new training application, we ran across an issue where when users would click a training session in the calendar, they'd be greeted with the dreaded Error in Application message.

Looking in the event log on the server, it threw this error:
 > Some part of your SQL statement is nested too deeply. Rewrite the query or break it up into smaller queries.

This appears to have been because this particular course was open to 150 users and was running into issues described [here](http://blogs.blackmarble.co.uk/blogs/rhepworth/post/2008/07/03/workflow-history-and-sql-error.aspx.).

The fix for us, as detailed in part 3, was to simply disable the site feature Office SharePoint Server Publishing.  We are using the site collection publishing features, but the site level ones were not needed.

If you're running the Enterprise SKU of Sharepoint and might have the potential for many workflows to be executed against a single list item, remember this tidbit.

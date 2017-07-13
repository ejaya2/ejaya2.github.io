---
layout: post
title: "Conditional Workflow Triggers in Nintex"
subtitle: "It can have unintended consequences"
date: 2017-07-13
category: Workflow
tags: [Nintex,Workflow,Blog]
author: Eric
---

In today's post, I wanted to go over a weird scenario that we encountered in one of our blog sites that has Nintex Workflow running over the top. It is a simple workflow that email blasts a group of users when the post is marked Approved. Pretty basic.

# The Set Up

As I mentioned above, the workflow was set to conditionally start on edit when the Approval Status was equal to approved. That looks  like this:

![Nintex Conditional Start Settings](http://ericjalexander.com/img/InitialConfiguration.JPG "Nintex Conditional Start Settings") 

This was working fine, until an edge case came into play.

# The Problem

The problem with this setup is that if you are using the social features, like Likes or Rating, the workflow will fire again when someone Likes the post. It took some investigation to figure this out though. When I went to the Posts list, I saw something like this:

![Social Interaction on the Posts List](http://ericjalexander.com/img/TheCulprit.JPG "Social Interaction on the Posts List")

What was curious about the workflow history was that it showed the workflow being run by the system account.

![System Account Executing the Workflow](http://ericjalexander.com/img/UnintendedExecution.JPG "System Account Executing the Workflow")

The only thing I could look at was this post had a Like on it. I tried digging through the social bowels of SharePoint via PowerShell but could not find a time stamp on when the Like was triggered. I figured it would be faster to just reporduce the workflow and test for myself. Sure enough, in this configuration the Like was enough to trigger the workflow even though the list item Modified metadata was not changing.

# The Fix

The fix was to change the workflow start condition for "Start when items are modified" to Yes.

![The Fix, Step 1](http://ericjalexander.com/img/TheFix_1.JPG "The Fix, Step 1")

The wrap the logic in a Run if action, configured to run as desired.

![The Fix, Step 2](http://ericjalexander.com/img/TheFix_2.JPG "The Fix, Step 2")

Publish the workflow, and you'll be all set.

# The Hypothesis

While I was PowerShelling my way through to find a cause to the errant execution, there are some metadata properties that get set when something is liked, LikedBy for one. My hypothesis is that Nintex sees this metadata change on the underlying item when a social interaction happens whereas the SharePoint system doesn't, as indicated by the Modified timestamp not changing. 

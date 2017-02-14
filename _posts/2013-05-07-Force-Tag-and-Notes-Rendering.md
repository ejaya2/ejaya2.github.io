---
layout: post
title: "Forcing Tags and Notes to Render"
subtitle: We really have to do this? 
date: 2013-05-07
category: Development
tags: [jQuery,Tags,Notes,Social]
author: Eric
---
One of the neat features of Sharepoint 2010 is the ability to add tags, notes, and likes to a page. The real fail to this is that if any information exists, it isn't immediately surfaced. You have to mouseover the icons to force a javascript function to fire, lame.

In a masterpage, you can simply add: 

`$("a[id^='TagsAndNotes_']").mouseover();`

to trigger the function and render the magenta background if comments/likes/tags exist.

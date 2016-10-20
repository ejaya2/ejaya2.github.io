---
layout: post
title: "Forcing Tags and Notes to Render"
date: 2013-05-07
---
<div><p>One of the neat features of Sharepoint 2010 is the ability to add tags, notes, and likes to a page. The real fail to this is that if any information exists, it isn't immediately surfaced. You have to mouseover the icons to force a javascript function to fire, lame.</p>
<p>In a masterpage, you can simply add <pre>$("a[id^='TagsAndNotes_']").mouseover(); </pre>to trigger the function and render the magenta background if comments/likes/tags exist.</p></div>

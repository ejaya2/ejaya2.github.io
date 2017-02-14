---
layout: post
title: Publishing HTML Column Frustrations
subtitle: Some field types make you want to pull your hair out
date: 2013-08-14
category: Development
tags: [jQuery,PublishingField]
author: Eric
---
Lately I've been doing a lot of work in the publishing/WCM area and one thing that's really aggravated me is that when you have a content type created for a page layout Sharepoint tries to be too cutesy when you edit the metadata for the item.

When you try to edit properties of the page, Sharepoint's awesome UX will find the first Publishing HTML field and set that as the focused item no matter where it is in the content type order. So if it's the last field in your content type, it steals the focus from the first field. This is horrible UX in that I want to fill my metadata from the top down, that's precisely why I ordered the columns the way I did.

Another side effect is that if you use Google Chrome and try to select a drop down list value while this field has focus, the drop down will autmatically collapse on you, again poor UX. This Page library is also using modal dialogs because if you don't, you can't properly use the rich html tools the field provides in Chrome either.

So I was looking for a fix. I came across a [post](http://sharepointchronicles.com/2011/11/sharepoint-2010-javascript-quirks/) from Rob mentioning this weird behavior but no real mention of a fix. In my Binging I also found this lovely [MSDN article](http://msdn.microsoft.com/en-us/library/sharepoint/microsoft.sharepoint.publishing.webcontrols.richhtmlfield.hasinitialfocus(v=office.14).aspx) that specifies there is an override for this lovely "feature". So I have to create a bunch of server side code to fix this poor UX? Surely there has to be a better option.

Since this editform already has some jQuery on it for other enhancements I thought I'd be able to fix this straight away no problem. Tried the typical document.ready things like `$("input[Title='Title']").click()` and `$("input[Title='Title']").focus()` to no avail. Turns out this information is loaded asynchronously. Yippee, time to dig around in the SOD javascript files.

After parusing through some of the on demand files and some trial and error, I finally came up with a hacky fix to get the focus out of the publishing html field and back up to the top of my form. Put this into a script tag:

```javascript
function waitUnitStuffIsLoaded() {
	setTimeout(function(){refocus()},1000);
}

function refocus(){
	$("input[Title='Title']").focus();
}
SP.SOD.executeOrDelayUntilScriptLoaded(waitUnitStuffIsLoaded, "sp.ui.spellcheck.js");
```

I tried a lot of the js files that were loaded on demand, like ribbon.js and this was the only one I found that worked.

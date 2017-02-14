---
layout: post
title: "Form Redirection"
subtitle: Ways of sending users to places after form submission
date: 2011-09-08
category: Development
tags: [Forms,jQuery]
author: Eric
---
I've seen quite a few posts around on various forums about redirecting users to a different page after filling out a form. I thought I'd recap some options. These may or may not be applicable to all SharePoint forms so review your options.

## The Easy OOTB Method
On your SharePoint site, be it a link added to the Quick Launch, stored in a links list, or created in a Content Editor web part, append a `?Source=` query string parameter to the end of the new form URL.  So something like this:
`http://myspsite.com/sites/mysite/Lists/Announcements/NewForm.aspx?Source=http://www.yahoo.com.`
This will redirect the user to Yahoo after completing the form. This could be any place, back to the default.aspx page or to a custom ThankYou page.

## A jQuery Method
I had asked a question on how to do this with jQuery on the old Stump the Panel forums on nothingbutsharepoint.com. [Matt Bramer](http://twitter.com/ionline247) was able to come up with a solution utilizing jQuery. This appends a source query string parameter to the URL for a hrefs that contain NewForm.aspx and returns them to the originating page.

```javascript
$(document).ready(function() {
  $("a[href*='NewForm.aspx']").click(function(){
    var curr_URL = document.location.href;
    appendTo_URL = "?Source=" + curr_URL;
    var link_URL = $(this).attr('href');
    //alert(link_URL + appendTo_URL);
    $(this).attr('href', link_URL + appendTo_URL);
  });
});
```

## A Custom Form Method
In SharePoint Designer, you can copy the list's NewForm.aspx page and create your own custom NewForm by closing the default web part and adding in a Custom ListForm. I've used [this Sharepoint Kings article](http://www.sharepointings.com/custom-list-forms) many times in situations where I'm heavily customizing a form and want to control the redirection.

## Other Interesting Methods
Alexander Bautz has an interesting method [here](http://sharepointjavascript.wordpress.com/2009/09/04/redirect-from-newform-to-editform-or-custom-page/).

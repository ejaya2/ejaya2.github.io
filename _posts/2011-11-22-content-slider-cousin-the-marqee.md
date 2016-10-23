---
layout: post
title: "The Content Slider's Cousin, the Marquee"
subtitle: But were't marquees like web circa 2000?
date: 2011-11-22
category: Development
tags: jQuery SPServices marquee slider
author: Eric
---
In my previous article, I showed you the framework for building your own content slider. Today, while perusing the [SharePoint forums](http://sharepoint.stackexchange.com/) on Stack Exchange, I found an interesting [post](http://sharepoint.stackexchange.com/questions/23775/list-driven-scrolling-marquee) that I thought I’d put to the test, creating a marquee with jQuery. Christophe has already tackled this with his great series on html calculated columns and an example can be seen [here](http://pathtosharepoint.com/Lists/TasksVisualization/AllItems.aspx). I thought since I had already done this in the past, utilizing that framework to create a similar output would be a snap.

This technique wouldn’t have the same type of scope like my previous article. This marquee could be used to display breaking news or special alerts and not full-fledged announcements.

To get started we need some of the usual suspects and one new library to add to our toolkit. Download, unzip and upload them to your scripts library.

 - [jQuery](http://jquery.com/")
 - [SPServices](http://spservices.codeplex.com)
 - [Marquee plugin](http://www.givainc.com/labs/marquee_jquery_plugin.htm)
 - Document library
 - Announcements list

To start off, we need to properly reference the resources:

```javascript
<script type="text/javascript" src="http://yourSPDomain/sites/site/library/jQuery.js"></script>
<script type="text/javascript" src="http://yourSPDomain/sites/site/library/SPServices.js"></script>
<script type="text/javascript" src="http://yourSPDomain/sites/site/library/jquery.marquee.min.js" ></script>
<link type="text/css" href=" http://yourSPDomain/sites/site/library/jquery.marquee.min.css">
```

Then we need to build up the markup to do populate the html structure that the marquee library needs. From the web site documentation, all that is needed is an unordered list. This makes the markup generation very simple. I'm reusing a good portion of the jQuery from my previous article so I'm not going to go into depth on what's happening.

```javascript
<script type="text/javascript">
$(document).ready(function (){
    var emptyResults = "<li>There are no current alerts.</li>"
    var toShow = false;
	  $().SPServices({
		operation: "GetListItems",
		async: false,
		listName: "Announcements",
		CAMLViewFields: "<ViewFields><FieldRef Name='Title' /><FieldRef Name='Body' /></ViewFields>",
		CAMLQuery: "<Query><OrderBy><FieldRef Name='Created' /></OrderBy>" +
			"<Where><Or><Geq><FieldRef Name='Expires' /><Value Type='DateTime'>" +
			"<Today /></Value></Geq><IsNull><FieldRef Name='Expires' />" +
			"</IsNull></Or></Where></Query>", 
		completefunc: function (xData, Status) {
		  var itemCount = $(xData.responseXML).find("[nodeName='rs:data']").attr("ItemCount");
		  if(itemCount > 0){
			toShow = true;
			$(xData.responseXML).find("[nodeName='z:row']").each(function() {
				var bodyHtml = "<li>" + $(this).attr("ows_Title");+ "</li>";
				$("#marquee").append(bodyHtml);
			});
		  }
		  else { 
			$("#marquee").append(emptyResults);
		  }
		}
	});
	
	if (toShow == true) {
		$("#marquee").marquee();
	}
}); 
	
</script>
<ul id="marquee" class="marquee" />
```

Instead of build a large html structure like before, the SPServices script is simply creating a list item (li) for each record returned. These get appended to the unordered list (ul) and thus the marquee is born. The plugin does allow for options to be passed into the function call to let you tweak how the marquee functions. You can read about those options on the [plugin page](http://www.givainc.com/labs/marquee_jquery_plugin.htm). 

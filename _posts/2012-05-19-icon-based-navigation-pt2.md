---
layout: post
title: "Creating an icon based navigation, part 2"
subtitle: Bringing the idea to life 
date: 2012-05-19
category: Development
tags: [Navigation,jQuery,SPServices]
author: Eric
---

In my [previous post](https://ejaya2.github.io/blog/2012/05/18/icon-based-navigation), I gave an overview of the problem and in this post my goal is to show you how to pull it off. The inspiration for this is my new favorite game, [Diablo 3](http://http//us.battle.net/d3/en/?-).

If you don't have an Asset Library, document library or picture library on your site, create one now. This is where the icons in the icon driven navigation will be stored.

Next create a custom list, in my example, it's called IconNav. Create the following columns:
 
 - image - Hyperlink or Picture, required, format URL as Picture
 - desc - Multiple lines of text, optional, plain text
 - link - &nbsp; Hyperlink or Picture, required, format URL as Hyperlink
 - order - Number, optional, 0 decimal places, (This column is purely optional and is used for specifying your own sort order)

 
![List Settings](http://ericjalexander.com/img/NavList.JPG "List Settings") 


Now upload all the icons you want to use into the previously mentioned document/picture/asset library.

![icons](http://ericjalexander.com/img//NavIconImages.JPG "Icons")


Now go to the IconNav list and start creating list items, filling in all the metadata.&nbsp; You'll end up with something that looks like this. Take note of this image as there are 4 items in the list. When you see the final solution, you'll notice there are only 3. That's because I removed anonymous access to the Angel link.

![IconNav list items](http://ericjalexander.com/img//NavListItems.JPG "IconNav list items")


Now the last part of this equation is to create a pretty display of all this information. To that we're turning to SharePoint Designer and creating a dataview web part. This is rudimentary rowview code so you can see what's going on, but any navigation you can think of could be output.

```html
<table border="0" cellspacing="0" width="100%" style="text-align:center; width:300px;">
	<tr>
		<td width="75%" class="ms-vb">
			<xsl:value-of select="@Title" /> 
		</td>  
	</tr>   
	<tr>     
		<td width="75%" class="ms-vb">
			<a href="http://pirateeric.sharepointspace.com{@link}" title="{@link.desc}"> 
				<img src="http://pirateeric.sharepointspace.com{@image}" alt="{@Title}" title="{@image.desc} image" style="border:0px;" /> 
			</a>  
		</td>   
	</tr>   
	<tr>     
		<td width="75%" class="ms-vb">  
			<xsl:value-of select="@desc" disable-output-escaping="yes" />  
		</td>   
	</tr>
	<tr>     
		<td width="75%" class="ms-vb">   
			<xsl:value-of select="@desc" disable-output-escaping="yes" /> 
		</td>  
	</tr>
</table>
```

This creates an output that looks like this.

![Much success](http://ericjalexander.com/img/Results.JPG "Much success")

Notice you only see 3 items, but there are 4 items in that nav list because anon users were removed. Ideally, permissions would match on the links and the corresponding pages so there aren't any security breaches.

I hope you've found this useful and a starting point to build your own icon based navigation. I know I'm excited to launch our icon driven solution in one of our web applications post 2010 upgrade.

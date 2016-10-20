---
layout: post
title: "AddWebPart Method of the WebPartPages Web Service"
date: 2013-01-17
---
It's been a bit of a hiatus since I've posted anything on my blog. 2 kids, a job change, and a move across country will do that. Anyway, tonight's post is one I'm doing more for self-preservation and hopefully others will find it useful. There are a lot of documentation black holes out there in the Sharepoint land and it seems that the WebPartPages web service is one of them.

In a project I'm currently working on, there is a need to automate a project creation process that is very manual to something more automated. Fortunately we have [Nintex Workflow](http://www.nintex.com/en-US/Products/Pages/Workflow.aspx) to handle situations like this.&#8203; Part of this workflow is to provision a new subsite after the project is approved by the project manager. Nintex makes this easy with their LazyApproval feature and create site action.

What I needed to do was upon site creation from a template, add some web parts to the home page. This is where I was running into the huge documentation gap. I turned to one of my goto libraries, [SPServices](http://spservices.codeplex.com), to protoype the actual calls before porting it over to my workflow. Fortunately someone had tried to do this same thing in the past and provided a working example for adding a content editor web part. That is described [here](http://spservices.codeplex.com/wikipage?title=AddWebPart&amp;referringTitle=WebPartPages) and worked no problem. My issue is I need to add list view web parts of document libraries and lists. I tried many things over the span of a couple days to tweak that to get my web parts onto the page. No dice.

Today I stumbled upon a [post](http://www.glynblogs.com/2011/04/exporting-the-xslt-list-view-web-part-in-sharepoint-2010.html) by [Glyn Clough](http://www.twitter.com/GlynClough) that filled that documentation black hole.

I need to use the List View Web Part markup, adjusting the ListUrl value were appropriate, like this for lists:
```
<?xml version="1.0" encoding="utf-8" ?>
<webParts>
 <webPart xmlns="http://schemas.microsoft.com/WebPart/v3">
  <metaData>
   <type name="Microsoft.SharePoint.WebPartPages.XsltListViewWebPart, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
   <importErrorMessage>Cannot import this Web Part.</importErrorMessage>
  </metaData>
  <data>
   <properties>
    <property name="ListUrl" type="string">Lists/CustomList</property>
    <property name="ExportMode" type="exportmode">All</property>
   </properties>
  </data>
 </webPart>
</webParts>
```
and like this for document libraries:
```
<?xml version="1.0" encoding="utf-8" ?>
<webParts>
 <webPart xmlns="http://schemas.microsoft.com/WebPart/v3">
  <metaData>
   <type name="Microsoft.SharePoint.WebPartPages.XsltListViewWebPart, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" />
   <importErrorMessage>Cannot import this Web Part.</importErrorMessage>
  </metaData>
  <data>
   <properties>
    <property name="ListUrl" type="string">Library</property>
    <property name="ExportMode" type="exportmode">All</property>
   </properties>
  </data>
 </webPart>
</webParts>
```
Once escaped and passed into my SPServices function I had web parts on my web part page.

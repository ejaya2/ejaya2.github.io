---
layout: post
title: "AddWebPart Method of the WebPartPages Web Service"
date: 2013-01-17
---
<p>It's been a bit of a hiatus since I've posted anything on my blog. 2 kids, a job change, and a move across country will do that. Anyway, tonight's post is one I'm doing more for self-preservation and hopefully others will find it useful. There are a lot of documentation black holes out there in the Sharepoint land and it seems that the WebPartPages web service is one of them.</p>
<p>In a project I'm currently working on, there is a need to automate a project creation process that is very manual to something more automated. Fortunately we have <a href="/web/20130419214243/http://www.nintex.com/en-US/Products/Pages/Workflow.aspx">Nintex Workflow</a> to handle situations like this.&#8203; Part of this workflow is to provision a new subsite after the project is approved by the project manager. Nintex makes this easy with their LazyApproval feature and create site action.</p>
<p>What I needed to do was upon site creation from a template, add some web parts to the home page. This is where I was running into the huge documentation gap. I turned to one of my goto libraries, <a href="/web/20130419214243/http://spservices.codeplex.com/">SPServices</a>, to protoype the actual calls before porting it over to my workflow. Fortunately someone had tried to do this same thing in the past and provided a working example for adding a content editor web part. That is described <a href="/web/20130419214243/http://spservices.codeplex.com/wikipage?title=AddWebPart&amp;referringTitle=WebPartPages">here</a> and worked no problem. My issue is I need to add list view web parts of document libraries and lists.&nbsp; I tried many things over the span of a couple days to tweak that to get my web parts onto the page. No dice.</p>
<p>Today I stumbled upon a <a href="/web/20130419214243/http://www.glynblogs.com/2011/04/exporting-the-xslt-list-view-web-part-in-sharepoint-2010.html">post</a> by <a href="/web/20130419214243/http://www.twitter.com/GlynClough">Glyn Clough</a> that filled that documentation black hole.</p>
<p>I need to use the List View Web Part markup like this for lists:</p>
<pre>&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;webParts&gt;
 &lt;webPart xmlns="http://schemas.microsoft.com/WebPart/v3"&gt;
  &lt;metaData&gt;
   &lt;type name="Microsoft.SharePoint.WebPartPages.XsltListViewWebPart, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" /&gt;
   &lt;importErrorMessage&gt;Cannot import this Web Part.&lt;/importErrorMessage&gt;
  &lt;/metaData&gt;
  &lt;data&gt;
   &lt;properties&gt;
    &lt;property name="ListUrl" type="string"&gt;Lists/CustomList&lt;/property&gt;
    &lt;property name="ExportMode" type="exportmode"&gt;All&lt;/property&gt;
   &lt;/properties&gt;
  &lt;/data&gt;
 &lt;/webPart&gt;
&lt;/webParts&gt;</pre>
<p>and like this for document libraries:</p>
<pre>&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;webParts&gt;
 &lt;webPart xmlns="http://schemas.microsoft.com/WebPart/v3"&gt;
  &lt;metaData&gt;
   &lt;type name="Microsoft.SharePoint.WebPartPages.XsltListViewWebPart, Microsoft.SharePoint, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" /&gt;
   &lt;importErrorMessage&gt;Cannot import this Web Part.&lt;/importErrorMessage&gt;
  &lt;/metaData&gt;
  &lt;data&gt;
   &lt;properties&gt;
    &lt;property name="ListUrl" type="string"&gt;Library&lt;/property&gt;
    &lt;property name="ExportMode" type="exportmode"&gt;All&lt;/property&gt;
   &lt;/properties&gt;
  &lt;/data&gt;
 &lt;/webPart&gt;
&lt;/webParts&gt;</pre>
<p>Once escaped and passed into my SPServices function I had web parts on my web part page.<br></p>
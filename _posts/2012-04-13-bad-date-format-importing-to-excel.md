---
layout: post
title: "Oddity with importing SharePoint data to Excel"
subtitle: Sometimes Excel is too helpful
date: 2012-04-13
category: Announcement
tags: Book
author: Eric
---
Imagine a scenario where you have a list of items with a product number or room number in the format of XX-XXXX. Now imaging you have an Excel workbook the pulls in data from this Sharepoint site including that field.

What you might notice is that Excel sees that it resembles a date and formats it as such for you. Handy, but not useful in this scenario. I tried a few solutions I found on Bing, like preformatting the destination field to text or general and putting an apostrophe before the data.

The apostrophe trick did work&#8203; if I was manually inputting the data, but would fail when doing a data refresh.

I eventually found a handy little setting in the Data Connection properties. Go into the data connection and click the Options link,

![Edit data connection](http://ericjalexander.com/img/editdataconn.PNG "Edit data connection")

From there, tick to enable the Disable date recognition option under Other Import Settings.

![Options](http://ericjalexander.com/img/options.PNG "Options")

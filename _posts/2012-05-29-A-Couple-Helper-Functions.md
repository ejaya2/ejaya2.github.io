---
layout: post
title: "A Couple Helper Functions"
date: 2012-05-29
---
<div>
</div>
<p>If you work with client side scripts and libraries like jQuery and <a href="/web/20130419214243/http://spservices.codeplex.com/">SPServices</a>, you'll often run into situations where the data you get back from.&nbsp; This isn't an issue with those libraries, they're just spitting back at you what SharePoint told them.&nbsp; The two areas where I see this the most in my dealings is with dates and person/group/lookup fields. As an update, I'm including something for number columns as well.<br></p>
<div>
</div>
<p>This post will show you one simple way to work with these columns and is also for my own sanity, as I feel I rewrite them almost every time I need them.&nbsp; These <strong>don't</strong> handle scenarios where multiple options are allowed as part of the column settings and they aren't optimized.&nbsp; <br></p>
<div>
</div>
<p></p>
<div>
</div>
<h2 class="ms-rteElement-H2">Person and Group and Lookup Columns&nbsp;</h2>
<div>
</div>
<p>When you get data back from SharePoint for one of these columns, it typically looks like this: 154;#555-1234 or 16;#Operations Department.&nbsp; Using the below function, I can get the descriptive value that SharePoint Designer seamless outputs in Dataview Webparts.</p>
<div>
</div>
<p style="margin-left:40px"></p>
<div>
</div>
<pre>function trimOWS(value){
 theString = value.indexOf(";#");
 theOutput = value.substring(theString +2);
 theLen = theOutput.length;
 if (theLen &lt; 8){theOutput = "Not Listed"}else{theOutput}
 return theOutput;
}
</pre>
<div>
Then in my SPServices function, I simply call it like:</div>
<div style="margin-left:40px"><pre>var edept = trimOWS($(this).attr("ows_EmployeeDepartment"));<br></pre></div>
<div>
</div>
<pre><br></pre>
<div>
This is testing against a string length of 8 as it's a double duty function, checking both department fields and phone numbers in my case.&nbsp; You could alter that to any value as needed.</div>
<div>&nbsp;</div>

<h2 class="ms-rteElement-H2">Date Fields&nbsp;</h2>
<p>Dates are another issue.&nbsp; SharePoint will spit back the date as 2012-05-29 12:00 (yyyy-mm-dd).&nbsp; There are several jQuery libraries out there for help in date formatting.&nbsp; I don't see the need to load an additional library to do that for you. This gives you flexibility to output a date in any format you wish with very little code using plain ole JavaScript.&nbsp;</p>
<p style="margin-left:40px"> <br></p>
<pre>function formatYear(value){
 var val = ""
 var mo = ""
 	if (value.substr(5, 2) &lt; 10){mo = value.substr(6, 1)}else{mo = value.substr(5, 2)}
 var dat = ""
 	if (value.substr(8, 2) &lt; 10 &amp;&amp; value.substr(8, 2) &gt; 0){dat = value.substr(9, 1)}else{dat = value.substr(8, 2)}
 var yr = value.substr(0, 4)
 var hr = value.substr(11,2)
	if (value.substr(11,2) &lt; 10){hr = value.substr(12,1)}else{hr = value.substr(11,2)} 	
 	if (hr &gt;= 12){ante = " PM"} else{ante = " AM"}
 var mi = value.substr(14,2)
 	if (value.substr(14, 2) &lt; 10){dt = value.substr(15, 1)}else{dt = value.substr(14, 2)}
	 
 val = mo + "/" + dat + "/" + yr + " " + hr + ":" + mi + ante;
 return val;
}
</pre>
<p></p>
<p>This uses the substring functions too bust the values apart and format a date and time in the fashion I want.&nbsp; It also has some logic in there to handle how to display values below 10 (personal preference, I don't like 05-29 but prefer 5-29) and AM/PM logic.&nbsp; This is called in the same fashion in my SPServices function:</p>
<p style="margin-left:40px"><br></p>
<pre><div> var start = formatYear($(this).attr("ows_StartDate"));</div></pre>
<h2 class="ms-rteElement-H2">Number Fields</h2>
<p>If you tell SharePoint to display a number column with 0 decimals, it displays it nicely in the UI.&nbsp; With the web services, you might get something like 16.0000000000.&nbsp; A simple way to handle that is with a replace regular expression. As an aside, regular expressions look hideous and make no real logical sense to me. I always seem to have to Bing my way to what I need or ask a collegue.&nbsp; For whole numbers, I use a formula like this:</p>
<p style="margin-left:40px"></p>
<pre> <div style="margin-left:40px">var seats = $(this).attr("ows_FilledSeats").replace(/\..*/g, "");</div></pre>

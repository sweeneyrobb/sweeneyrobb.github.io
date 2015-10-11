---
title: SharePoint 2013 - Discovering Timeline View Automation
layout: post
description:
categories: [blog]
tags: [sharepoint]
comments: true
modified: 2014-10-07T12:15:25-08:00
---

One of more subtle but great little additions to SharePoint 2013 is the Task Timeline. This is most noticeable when creating a site using the “Project Site” template. During various events within a particular project lifecycle, tasks may be created dynamically. I thought it would be great to automatically populate the timeline with these auto-generated task...boy I didn’t realize what I was getting myself into.

![Screenshot of SharePoint 2013 Timeline View control.](/images/20140209-001.png)

There is little to no documentation regarding the technical nature of the Timeline component on the web.  Some posts with people have similar issues as myself and looking for guidance. I still felt this shouldn’t be a terribly difficult task. I started poking around with some the IE Developer Tools. I found code executing from ‘sp.ui.timeline.js’.  At this point, I figured I would just read some JavaScript and figure out what I needed to do…wrong! This file is obfuscated. Spent longer than I probably should have, reasoning the obfuscated JavaScript.  Even though I did learn a lot, I decided to tackle this a different way.

I popped open Fiddler2 and monitored a simple page load. After interrogating the various requests, I found one that looked like my ticket.

    <?XML:NAMESPACE PREFIX="[default] http://schemas.microsoft.com/sharepoint/clientquery/2009"
        NS="http://schemas.microsoft.com/sharepoint/clientquery/2009" />

    <request xmlns="http://schemas.microsoft.com/sharepoint/clientquery/2009" schemaversion="15.0.0.0"
        libraryversion="15.0.0.0" applicationname="Javascript Library">
    	<actions>
    		...
    		<query id="12" objectpathid="10">
    			<query selectallproperties="true">
    				<properties></properties>
    			</query>
    		</query>
    	</actions>
    	<objectpaths>
    		<staticproperty id="0" typeid="{3747adcd-a3c3-41b9-bfab-4a64dd2f1e0a}" name="Current" />
    		<property id="2" name="Web" parentid="0"></property>
    		<property id="4" name="Lists" parentid="2"></property>
    		<method id="6" name="GetById" parentid="4">
    			<parameters>
    				<parameter type="Guid">{1293ee92-1074-4687-93d3-516fb8a0b3ac}</parameter>
    			</parameters>
    		</method>
    		<property id="8" name="RootFolder" parentid="6"></property>
    		<property id="10" name="Properties" parentid="8"></property>
    	</objectpaths>
    </request>

The request looks pretty straight-forward.  Get the current context, web, list (byID), RootFolders, Properties, then action to select All Properties. The response was as expected. I had all the contents of the property bag in JSON format.  Two properties jump out immediately (“TimelineDefaultView” and “Timeline_Timeline”).

    [{
    	...
    	12, {
    		...
    		"TimelineDefaultView" : "Timeline",
    		"Timeline_Timeline" : "&lt;TLViewData&gt;...&lt;\TLViewData&gt;",
    	}
    }]

Looking at this data, I made some pretty large assumptions. The Timeline supports multiple views. Each view is prefixed with ”Timeline_”. Acting on these assumptions, I took the value of the Timeline_Timeline entry, decoded and formatted. After review it looks like the jackpot. While very shorthand we can see Milestones, Callouts, Tasks, Options and formatting.

    <tlviewdata>
      <!-- Item Format Set -->
      <fmtset>
        <fmt id="0" type="0" t2="1" t1="0" thm="0001" clr="FFEE2222" />
      </fmtset>
      <!-- Callout Set -->
      <fltset>
        <ft id="{00000000-0000-0000-0000-000000000000}" h="20" x="0" y="4294967282" fmt="1" uid="4294967295" uidsrc="1" ontl="0" />
      </fltset>
      <!-- Bar Element Set -->
      <tskset>
        <t id="{00000000-0000-0000-0000-000000000000}" fmt="0" uid="4294967295" ch="4294967295" uidsrc="1" ontl="0" />
      </tskset>
      <!-- TimelineOptions Set -->
      <options dateformat="255" panzoomt="9" projsummfmt="3" showdates="1" showprojsummdates="1" showtoday="1" showts="1" timelineheight="110" timelinewidth="-1" timescalet="8" todayt="10" />
      <!-- Milestone Set -->
      <mlset>
        <m id="{00000000-0000-0000-0000-000000000000}" x="0" y="35" fmt="2" uid="4294967295" uidsrc="1" ontl="0" />
      </mlset>
      <!-- Text Style Set -->
      <txtset>
        <style id="0" type="0" thm="0001" clr="FFEE2222" strk="0" und="0" ital="0" bold="0" font="Segoe UI" sz="8">
      </txtset>
    </tlviewdata>

Seemed like a rather large bit of xml data to store in a property bag. Made me think—why store information this way, rather than on each item directly? My assumption is, doing it this way, likely prevents any event handlers from being triggered, when simply modifying what items are appearing in the view. One step better might be to store it in JSON format; A bit more concise and likely easier to deal with from browser.  If I recall, looking at the obfuscated “sp.ui.timeline.js” there was quite a bit of code to serialize the JavaScript objects into XML.

At any rate, now that I have some sample data, I copied the data and pasted the code into some a .cs file using the "Paste Special", "Paste XML As Classes". I found this worked a little better than using xsd.exe to take XML –> XSD –> CS (xsd.exe had generated more unboxing code :T ). I had to fix a little bit of data typing but easier than typing up all the classes by hand.

![Image of Visual Studio's 'Edit', 'Paste Special', 'Paste XML As Classes' menu.](/images/20140209-002.png)

Now that I had C# classes, I can de-serialize, manipulate, serialize and store back into SharePoint. I’m a bit lazy, so I have leveraged .NET server side (was already in a provider hosted solution). I would have rather liked to have done this on the client, but, that means I have to find a good library that can serialize/de-serialize JavaScript objects from/to XML (in browser). If anyone knows of a good one, I would greatly appreciate a link Smile

All this headache for doing something, that should be, so simple. I hope I have, at the very least, saved someone the work when automating the creation of Timeline views. I’ll try to follow up with a commented XML document and/or a set of documented classes with XML serialization attributes.

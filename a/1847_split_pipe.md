<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- https://github.com/jeremytammik/Pipe-Split
  /a/src/rvt/Pipe-Split/

- split a conduit
  https://forums.autodesk.com/t5/revit-api-forum/trouble-of-my-own-implementation-of-breakcurve-on-conduit/m-p/8426899/highlight/false
  16561381 [Programatically Place Conduit Fitting]

- split pipe into lengths
  14201186 [Need to call command Split Element (SL) with defined filter and length]


twitter:

Tutorial on splitting a pipe, calling the SL split element command and splitting a conduit in the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon https://bit.ly/splitpipetutor

Today, let's focus on splitting pipes and other things, starting with a nicely structured tutorial
&ndash; Abdelaziz' split pipe tutorial
&ndash; Calling the SL split element command
&ndash; Splitting a conduit...

linkedin:

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="600"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### Split Pipe Tutorial and Related Cases

Today, let's focus on splitting pipes and other things, starting with a nicely structured tutorial shared
by Abdelaziz [Zizobiko25](https://github.com/Zizobiko25) Fadoul:

- [Abdelaziz' split pipe tutorial](#3)
- [Calling the SL split element command](#4)
- [Splitting a conduit](#5)
- [George Floyd in Memoriam](#6)

####<a name="3"></a> Abdelaziz' Split Pipe Tutorial

Hi folks,

Hope you are all safe and keeping well.

As there is no explicit SDK sample for splitting pipe instances into standard lengths without using the fabrication configurations, so I thought to share with you the below snippet. This is built upon a contribution that I came across on this platform, and uses functions that are developed on other samples (e.g. Find connected element, etc.), however, I thought to enhance it and explain the steps to achieve such a task. The fundamental idea for this is to delete the selected pipe and replace it with the creation of 2 pipes instead. 

1-	Establish a new transaction: As this is going to modify the Revit document, then we need to have it done within a transaction context.

<pre class="code">
  <span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;tx&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;activeDoc.Document&nbsp;)&nbsp;)
  {
  &nbsp;&nbsp;tx.Start(&nbsp;<span style="color:#a31515;">&quot;split&nbsp;pipe&quot;</span>&nbsp;);
  &nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;systemtype&nbsp;=&nbsp;system.GetTypeId();
  &nbsp;&nbsp;SplitPipe(&nbsp;pipes[&nbsp;0&nbsp;],&nbsp;system,&nbsp;activeDoc,&nbsp;systemtype,&nbsp;pipeType&nbsp;);
  &nbsp;&nbsp;tx.Commit();
  }
</pre>

2-	We then obtain the selected pipe basic data including its length, associated level Id and system Id.

<pre class="code">
&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;levelId&nbsp;=&nbsp;segment.get_Parameter(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.RBS_START_LEVEL_PARAM&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;  .AsElementId();
 
&nbsp;&nbsp;<span style="color:green;">//&nbsp;system.LevelId;</span>
&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;systemtype&nbsp;=&nbsp;_system.GetTypeId();
 
&nbsp;&nbsp;<span style="color:green;">//&nbsp;selecting&nbsp;one&nbsp;pipe&nbsp;and&nbsp;taking&nbsp;its&nbsp;location.</span>
&nbsp;&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;c1&nbsp;=&nbsp;(segment.Location&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">LocationCurve</span>).Curve;&nbsp;
 
&nbsp;&nbsp;<span style="color:green;">//Pipe&nbsp;diameter</span>
&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;pipeDia&nbsp;=&nbsp;<span style="color:#2b91af;">UnitUtils</span>.ConvertFromInternalUnits(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;segment.get_Parameter(&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.RBS_PIPE_DIAMETER_PARAM&nbsp;).AsDouble(),
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">DisplayUnitType</span>.DUT_MILLIMETERS&nbsp;);  
</pre>

3-	We then compare the obtained length, including the fitting length, with the standard length that we want to assign (for instance, 6m here).
If it is longer than that, then we proceed. 

<pre class="code">
  <span style="color:green;">//Standard&nbsp;length</span>
  <span style="color:blue;">double</span>&nbsp;l&nbsp;=&nbsp;6000;
   
  <span style="color:green;">//Coupling&nbsp;length</span>
  <span style="color:blue;">double</span>&nbsp;fittinglength&nbsp;=&nbsp;(1.1&nbsp;*&nbsp;pipeDia&nbsp;+&nbsp;14.4);
   
  <span style="color:green;">//&nbsp;finding&nbsp;the&nbsp;length&nbsp;of&nbsp;the&nbsp;selected&nbsp;pipe.</span>
  <span style="color:blue;">double</span>&nbsp;len&nbsp;=&nbsp;<span style="color:#2b91af;">UnitUtils</span>.ConvertFromInternalUnits(&nbsp;segment.get_Parameter(&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.CURVE_ELEM_LENGTH&nbsp;).AsDouble(),&nbsp;<span style="color:#2b91af;">DisplayUnitType</span>.DUT_MILLIMETERS&nbsp;);
   
  <span style="color:blue;">if</span>(&nbsp;len&nbsp;&lt;=&nbsp;l&nbsp;)
  &nbsp;&nbsp;<span style="color:blue;">return</span>;
</pre>

4-	We then determine the splitting point by taking the fracture between the required length to the total length.

<pre class="code">
  <span style="color:blue;">var</span>&nbsp;startPoint&nbsp;=&nbsp;c1.GetEndPoint(&nbsp;0&nbsp;);
  <span style="color:blue;">var</span>&nbsp;endPoint&nbsp;=&nbsp;c1.GetEndPoint(&nbsp;1&nbsp;);
   
  <span style="color:#2b91af;">XYZ</span>&nbsp;splitpoint&nbsp;=&nbsp;(endPoint&nbsp;-&nbsp;startPoint)&nbsp;*&nbsp;(l&nbsp;/&nbsp;len);
   
  <span style="color:blue;">var</span>&nbsp;newpoint&nbsp;=&nbsp;startPoint&nbsp;+&nbsp;splitpoint;
   
  <span style="color:#2b91af;">Pipe</span>&nbsp;pp&nbsp;=&nbsp;segment&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Pipe</span>;
   
  <span style="color:green;">//&nbsp;Find&nbsp;two&nbsp;connectors&nbsp;which&nbsp;pipe&#39;s&nbsp;two&nbsp;ends&nbsp;connector&nbsp;connected&nbsp;to.&nbsp;</span>
  <span style="color:#2b91af;">Connector</span>&nbsp;startConn&nbsp;=&nbsp;FindConnectedTo(&nbsp;pp,&nbsp;startPoint&nbsp;);
  <span style="color:#2b91af;">Connector</span>&nbsp;endConn&nbsp;=&nbsp;FindConnectedTo(&nbsp;pp,&nbsp;endPoint&nbsp;);
</pre>

5-	We are then able to create the first and second pipe segments using the command create:

<pre class="code">
  <span style="color:green;">//&nbsp;creating&nbsp;first&nbsp;pipe&nbsp;</span>
  <span style="color:#2b91af;">Pipe</span>&nbsp;pipe&nbsp;=&nbsp;<span style="color:blue;">null</span>;
  <span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;_pipeType&nbsp;)
  {
  &nbsp;&nbsp;pipe&nbsp;=&nbsp;<span style="color:#2b91af;">Pipe</span>.Create(&nbsp;_activeDoc.Document,&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;_pipeType.Id,&nbsp;levelId,&nbsp;startConn,&nbsp;newpoint&nbsp;);
  }
   
  <span style="color:#2b91af;">Connector</span>&nbsp;conn1&nbsp;=&nbsp;FindConnector(&nbsp;pipe,&nbsp;newpoint&nbsp;);
   
  <span style="color:green;">//Check&nbsp;+&nbsp;fitting</span>
  <span style="color:#2b91af;">XYZ</span>&nbsp;fittingend&nbsp;=&nbsp;(endPoint&nbsp;-&nbsp;startPoint)&nbsp;
  &nbsp;&nbsp;*&nbsp;((l&nbsp;+&nbsp;(fittinglength&nbsp;/&nbsp;2))&nbsp;/&nbsp;len);
   
  <span style="color:green;">//New&nbsp;point&nbsp;after&nbsp;the&nbsp;fitting&nbsp;gap</span>
  <span style="color:blue;">var</span>&nbsp;endOfFitting&nbsp;=&nbsp;startPoint&nbsp;+&nbsp;fittingend;
   
  <span style="color:#2b91af;">Pipe</span>&nbsp;pipe1&nbsp;=&nbsp;<span style="color:#2b91af;">Pipe</span>.Create(&nbsp;_activeDoc.Document,&nbsp;
  &nbsp;&nbsp;systemtype,&nbsp;_pipeType.Id,&nbsp;levelId,&nbsp;endOfFitting,&nbsp;
  &nbsp;&nbsp;endPoint&nbsp;);
   
  <span style="color:green;">//&nbsp;Copy&nbsp;parameters&nbsp;from&nbsp;previous&nbsp;pipe&nbsp;to&nbsp;the&nbsp;following&nbsp;Pipe.&nbsp;</span>
  CopyParameters(&nbsp;pipe,&nbsp;pipe1&nbsp;);
  <span style="color:#2b91af;">Connector</span>&nbsp;conn2&nbsp;=&nbsp;FindConnector(&nbsp;pipe1,&nbsp;endOfFitting&nbsp;);
  _&nbsp;=&nbsp;_activeDoc.Document.Create.NewUnionFitting(&nbsp;conn1,&nbsp;conn2&nbsp;);
   
  <span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;endConn&nbsp;)
  {
  &nbsp;&nbsp;<span style="color:#2b91af;">Connector</span>&nbsp;pipeEndConn&nbsp;=&nbsp;FindConnector(&nbsp;pipe1,&nbsp;endPoint&nbsp;);
  &nbsp;&nbsp;pipeEndConn.ConnectTo(&nbsp;endConn&nbsp;);
  }
</pre>

6-	Once this is done successfully, we can then delete the original pipe segment

<pre class="code">
  <span style="color:#2b91af;">ICollection</span>&lt;<span style="color:#2b91af;">ElementId</span>&gt;&nbsp;deletedIdSet&nbsp;
  &nbsp;&nbsp;=&nbsp;_activeDoc.Document.Delete(&nbsp;segment.Id&nbsp;);
   
  <span style="color:blue;">if</span>(&nbsp;0&nbsp;==&nbsp;deletedIdSet.Count&nbsp;)
  {
  &nbsp;&nbsp;<span style="color:blue;">throw</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Exception</span>(&nbsp;<span style="color:#a31515;">&quot;Deleting&nbsp;the&nbsp;selected&nbsp;elements&nbsp;in&nbsp;Revit&nbsp;failed.&quot;</span>&nbsp;);
  }
</pre>

Link to the full code is accessible from
the [Pipe-Split GitHub repository](https://github.com/Zizobiko25/Pipe-Split).

Hope this useful for someone. Give me a shout if you have any questions.

Thanks, Abdelaziz

Many thanks to *You*, Abdelaziz, for putting together and sharing this very nice and clean solution and helpful explanation!


####<a name="4"></a> Calling the SL Split Element Command

While on the topic of splitting things, here is another related old case, 14201186 *Need to call command Split Element (SL) with defined filter and length*:

**Question:** I wanted to call the Split Element (SL) command through API and wants to give the filter to select only Pipe with predefined length to split.

Can you please help us with this?

**Answer:** You can call many built-in Revit commands using
the [`PostCommand` API](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.3).

However, this just launches the built-in command as is, prompting the user for all further input and other interaction.

Therefore, if you wish to drive this completely automatically, it will be hard to make use of the built-in command.

<!--
obsolete:

Here is a discussion of the basics of [creating and modifying pipes through the API](

http://knowledge.autodesk.com/search-result/caas/CloudHelp/cloudhelp/2015/ENU/Revit-API/files/GUID-0B91BAE8-59FD-4D4B-84E6-53B6A21DE00A-htm.html?_ga=2.49832166.1581756714.1525612885-2130181328.1465883366
-->

The specific question of splitting a pipe into shorter segments of a given length has been discussed repeatedly in the Revit API discussion forum:

- [Split a pipe at a specific lenght](https://forums.autodesk.com/t5/revit-api-forum/split-a-pipe-at-a-specific-lenght/m-p/5533100)
- [Pipe / Duct spliting using the Revit API](https://forums.autodesk.com/t5/revit-api/pipe-duck-spliting-using-the-revit-api/m-p/3312303)
- [Split Ducts with Union](https://forums.autodesk.com/t5/revit-api/split-ducts-with-union/m-p/5489135)


####<a name="5"></a> Splitting a Conduit

Another more recent related case is 16561381 *Programmatically Place Conduit Fitting*, on splitting a conduit:

**Question:** We would like to know how to programmatically place a conduit fitting at a certain location on a piece of conduit.
Basically, we want to exactly what this tool does (screenshots included as well) but we do not want the user to have to click where they place it &ndash; we want to programmatically control it's placement on a piece of conduit.

<center>
<img src="img/place_conduit_fitting_1.png" alt="Place conduit fitting" title="Place conduit fitting" width="529"/> <!-- 529 -->
<img src="img/place_conduit_fitting_2.png" alt="Place conduit fitting" title="Place conduit fitting" width="190"/> <!-- 190 -->
</center>

We have been unable to find anything within the API or forums which accomplish this. Please advise on how this can be done.

Tool help info:

- [Add Conduit Fittings](https://help.autodesk.com/view/RVT/2019/ENU/?guid=GUID-12A049FB-F779-4184-97A9-2E700945DE66)

**Answer:** I believe that one possible approach to achieve this would be to place the fitting in the appropriate position in the model first using `NewFamilyInstance`, and then connect it to the neighbouring conduits.
I would expect them to adjust automatically.

I discovered and demonstrated that approach in the series of posts
on [implementing a rolling offset](http://thebuildingcoder.typepad.com/blog/2014/01/final-rolling-offset-using-pipecreate.html).

To make sure I am not missing anything, I am checking with the development team for you as well.

Meanwhile, here are some other discussions on placing similar fittings:

- [Cable Tray Orientation and Fittings](http://thebuildingcoder.typepad.com/blog/2010/05/cable-tray-orientation-and-fittings.html)
- [Use of NewTakeOffFitting on a Duct](http://thebuildingcoder.typepad.com/blog/2011/02/use-of-newtakeofffitting-on-a-duct.html)
- [Set Elbow Fitting Type](http://thebuildingcoder.typepad.com/blog/2011/02/set-elbow-fitting-type.html)
- [Use of NewTakeOffFitting on a Pipe](http://thebuildingcoder.typepad.com/blog/2011/04/use-of-newtakeofffitting-on-a-pipe.html)
- [Simpler Rolling Offset Using NewElbowFitting](http://thebuildingcoder.typepad.com/blog/2014/01/newelbowfitting-easily-places-rolling-offset-elbow-fittings.html)
- [NewCrossFitting Connection Order](http://thebuildingcoder.typepad.com/blog/2014/10/newcrossfitting-connection-order.html)

The development team added some more detailed advice:

You can use `Document.Create.NewElbowFitting` taking two `Connector` arguments.

Depending on the orientation of the MEP Curves it might fail and then require a Transition or a Union instead.

If there are three connectors, it requires a Tee; if there are four connectors, it requires a Cross.

It uses the family routing settings to assign the type and the `size_lookup` in the fitting family to determine the geometrical representation of the FamilyInstance.

So, in case of different diameters to connect, it will create other objects in between, such as transitions (e.g., for pipes or ducts) or a junction box (e.g., for conduits).

**Response:** We are already doing something similar, but it seems wildly inefficient, and I was hoping the API provided something we were missing as Pipe (and other curve based families) seem to have 'special' methods that conduit lacks.

[Trouble of my own implementation of BreakCurve on Conduit](https://forums.autodesk.com/t5/revit-api-forum/trouble-of-my-own-implementation-of-breakcurve-on-conduit/m-p/8426899)

To be sure, we did research this thoroughly but as noted above our implementation has a 'clunky' feel to it when compared to the ease of the fitting tool in the UI (although watching updater messages shows Revit may be handling it in a similar way we are). We are under a time crunch at the moment and hope this methodology will ease some of the 'heartburn' we are experiencing with fittings on sloped conduit as well.

**Answer:** Yes, it does indeed look as if the Revit API forum thread you pointed out covers it pretty well.

I'm afraid I have nothing constructive to add to that, and, as you say, Revit probably does something similar internally as well.

####<a name="6"></a> George Floyd in Memoriam

"It's my face man
<br/>I didn't do nothing serious man
<br/>please
<br/>please
<br/>please I can't breathe
<br/>please man
<br/>please somebod
<br/>please man
<br/>I can't breathe
<br/>I can't breathe
<br/>please
<br/>(inaudible)
<br/>man can't breathe, my face
<br/>just get u
<br/>I can't breathe
<br/>please (inaudible)
<br/>I can't breathe sh*t
<br/>I will
<br/>I can't move
<br/>mama
<br/>mama
<br/>I can't
<br/>my knee
<br/>my nuts
<br/>I'm through
<br/>I'm through
<br/>I'm claustrophobic
<br/>my stomach hurt
<br/>my neck hurts
<br/>everything hurts
<br/>some water or something
<br/>please
<br/>please
<br/>I can't breathe officer
<br/>don't kill me
<br/>they gon' kill me man
<br/>come on man
<br/>I cannot breathe
<br/>I cannot breathe
<br/>they gon' kill me
<br/>they gon kill me
<br/>I can't breathe
<br/>I can't breathe
<br/>please sir
<br/>please
<br/>please
<br/>please I can't breathe"

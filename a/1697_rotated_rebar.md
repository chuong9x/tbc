<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- https://forums.autodesk.com/t5/revit-api-forum/relative-path-for-folder/m-p/8349026

- https://forums.autodesk.com/t5/revit-api-forum/revit-2019-tooltip-expandedvideo-does-not-work/m-p/8346705
  All Revit 2019 tooltip videos are in MP4 format now. See under: "C:/Program Files/Autodesk/Revit 2019/videos/"

- 14715374 [How to read/write Bolt, Plate & Weld using Revit API]

- https://forums.autodesk.com/t5/revit-api-forum/rotation-of-stirrup-rebar-rotated-beam-api/m-p/8251906
  https://forums.autodesk.com/t5/revit-structure-forum/rotation-of-stirrup-rebar-rotated-beam/m-p/8250053

Creating rotated rebar stirrups with the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon

I'll start this week with several solutions from the Revit API discussion forum and elsewhere, especially two different approaches to create rotated rebar stirrups
&ndash; Embedded tooltip icon resource
&ndash; Revit 2019 tooltip videos are MP4
&ndash; How to read and write bolt, plate and weld
&ndash; Creating rotated rebar stirrups...

-->

### Resources, Rotated Rebar Stirrups and Steel Stuff

I'll start this week with several solutions from 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) and
elsewhere, especially two different approaches to create rotated rebar stirrups:

- [Embedded tooltip icon resource](#2) 
- [Revit 2019 tooltip videos are MP4](#3) 
- [How to read and write bolt, plate and weld](#4) 
- [Creating rotated rebar stirrups](#5) 

<center>
<img src="img/rebar_in_rotated_beam.png" alt="Rebar in rotated beam" width="138">
</center>


#### <a name="2"></a> Embedded Tooltip Icon Resource

Let's start with a quick question on
the ‎[relative path for folder](https://forums.autodesk.com/t5/revit-api-forum/relative-path-for-folder/m-p/8349026) in order to load an icon image for an external command ribbon button:

**Question:** I am starting with a first Revit add in and one thing I am concerned is how to create the relative path for my icon URL as in the following code:

<pre class="code">
&nbsp;&nbsp;<span style="color:green;">//&nbsp;Determine&nbsp;the&nbsp;data&nbsp;for&nbsp;a&nbsp;push&nbsp;button,&nbsp;</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;include&nbsp;the&nbsp;name&nbsp;of&nbsp;the&nbsp;command&nbsp;to&nbsp;execute</span>
&nbsp;&nbsp;<span style="color:#2b91af;">PushButtonData</span>&nbsp;pushButtonData51
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">PushButtonData</span>(&nbsp;<span style="color:#a31515;">&quot;DetachModels&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;DetachModels&quot;</span>,&nbsp;thisAssemblyPath,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;CherryBIMservices.BatchDetachedModel&quot;</span>&nbsp;);
 
&nbsp;&nbsp;<span style="color:green;">//Add&nbsp;push&nbsp;button&nbsp;to&nbsp;the&nbsp;ribbon&nbsp;panel</span>
&nbsp;&nbsp;<span style="color:#2b91af;">PushButton</span>&nbsp;pushButton51&nbsp;=&nbsp;ManagePanel.AddItem(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;pushButtonData51&nbsp;)&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">PushButton</span>;
 
&nbsp;&nbsp;<span style="color:green;">//&nbsp;Add&nbsp;a&nbsp;tool&nbsp;tip&nbsp;for&nbsp;a&nbsp;button</span>
&nbsp;&nbsp;pushButton51.ToolTip&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Batch&nbsp;detach&nbsp;models&quot;</span>;
 
&nbsp;&nbsp;<span style="color:green;">//&nbsp;Add&nbsp;URL&nbsp;of&nbsp;image,&nbsp;set&nbsp;as&nbsp;Bitmap&nbsp;and&nbsp;set&nbsp;</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;the&nbsp;image&nbsp;as&nbsp;the&nbsp;icon&nbsp;for&nbsp;the&nbsp;pushbutton</span>
&nbsp;&nbsp;<span style="color:#2b91af;">Uri</span>&nbsp;uriImage51&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Uri</span>(&nbsp;<span style="color:maroon;">@&quot;C:\ProgramData\Autodesk\Revit\Addins\2017\CherryBIMservices\51.png&quot;</span>&nbsp;);
&nbsp;&nbsp;BitmapImage&nbsp;bitmapImage51&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;BitmapImage(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;uriImage51&nbsp;);
&nbsp;&nbsp;pushButton51.LargeImage&nbsp;=&nbsp;bitmapImage51;
</pre>

My question is how to replace the absolute path to the icon file:

- C:\ProgramData\Autodesk\Revit\Addins\2017\CherryBIMservices\51.png

I would like to use a relative path instead.

Thank you so much for your help.

**Answer:** I would recommend embedding your icon as an embedded resource.

Here are two out of many related discussions of the topic by The Building Coder:

- [Retrieve embedded resource image](http://thebuildingcoder.typepad.com/blog/2012/06/retrieve-embedded-resource-image.html)
- [Scaling a bitmap for the large and small image icons](http://thebuildingcoder.typepad.com/blog/2018/05/scaling-a-bitmap-for-the-large-and-small-image-icons.html)

Then you will have no need for any file path at all, and life will be easy and sweet.


#### <a name="3"></a> Revit 2019 Tooltip Videos are MP4

Next, a quick explanation why ‎[Revit 2019 tooltip ExpandedVideo does not work](https://forums.autodesk.com/t5/revit-api-forum/revit-2019-tooltip-expandedvideo-does-not-work/m-p/8346705):

**Question:** I am migrating my existing plugin from Revit 2018 to Revit 2019. I see that the animation (.swf file) in the tool tip does not play. This works perfect with 2018 but not with 2019. The message says:

<center>
<img src="img/tooltip_video_could_not_be_played.png" alt="Tooltip video could not be played" width="353">
</center>

The flash version is 29.0.0171 (Latest Version). Are there any API changes in this section? Is it an issue with the Flash version?

**Answer:** All Revit 2019 tooltip videos are in MP4 format now.

You can see for yourself under:

- C:/Program Files/Autodesk/Revit 2019/videos


#### <a name="4"></a> How to Read and Write Bolt, Plate and Weld

Here is a simple question that was not raised in the discussion forum and therefore is especially important to share here, at least:

**Question:** Revit added native Bolt, Plate & Weld in version 2019, but I can find related API to read/write them. Please advise where I can find them, and provide a sample on how to use them if possible.

**Answer:** You can find samples about how to create and read steel elements such as plates, bolts, welds, etc., in the Revit SDK sample:

- RevitSDK\Software Development Kit\Samples\SampleCommandsSteelElements

I hope this helps.

**Response:** Yes. It does. Thank you.


#### <a name="5"></a> Creating Rotated Rebar Stirrups

Now for the main topic for today:

**Question:** I have an issue creating stirrups in rotated beams.

I originally described it in the thread
on [rotation of stirrup rebar in a rotated beam](https://forums.autodesk.com/t5/revit-structure-forum/rotation-of-stirrup-rebar-rotated-beam/m-p/8250053) in
the Revit structure forum, but it is in fact an API issue.

Here is an image showing the corrupted layout on a 30 degrees rotated beam &ndash; part of the rebar near the centre and outside:

<center>
<img src="img/rotated_rebar_stirrup_3Dview-right-30.png" alt="Corrupted layout on a 30 degrees rotated beam" width="500">
</center>

The correct layout looks like this:

<center>
<img src="img/rotated_rebar_stirrup_different-template-layout-OK.png" alt="Layout OK" width="340">
</center>

**Answer:** I opened the RVT that was attached in the tread.

It generated many debug warnings about non-conformal matrix used in the Rebar element.

Probably, this happens because the rebar was created incorrectly, e.g., passing wrong arguments for `origin`, `xVec` and `yVec`.

I attached a [TestCommand.cs file](zip/rebar_stirrup_rotation_test_command.cs) which achieves the job in two ways:

First solution is using the same functions that the customer used: `Rebar.CreateFromRebarShape` and `rebar.ScaleToBox`.

Here is an image to help understand what arguments you should pass:

<center>
<img src="img/rebar_stirrup_rotation_first_solution.png" alt="CreateFromRebarShape arguments" width="500">
</center>

It is very important how you pass `origin`, `xVec` and `yVec`, because this data is used to create the bar position transform.

It is a transformation from family coordinates (`origin` (0,0,0), `xVec` (1,0,0), `yVec` (0,1,0)) to the new position in the model (`origin`, `xVec` and `yVec`).

For the second solution, I used the `Rebar.CreateFromCurves` function.

Here is the complete code:

<pre class="code">
<span style="color:blue;">using</span>&nbsp;System;
<span style="color:blue;">using</span>&nbsp;System.Collections.Generic;
 
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.DB;
 
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.UI;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.UI.Selection;
 
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.Attributes;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.DB.Structure;
<span style="color:blue;">using</span>&nbsp;System.Windows.Forms;
 
<span style="color:blue;">namespace</span>&nbsp;TestRebar
{
&nbsp;&nbsp;[<span style="color:#2b91af;">Transaction</span><span style="color:#2b91af;">Attribute</span>(&nbsp;<span style="color:#2b91af;">TransactionMode</span>.Manual&nbsp;)]
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">TestRebar</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IExternalCommand</span>
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UIApplication</span>&nbsp;m_uiApp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;m_doc;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;elementId&nbsp;=&nbsp;<span style="color:#2b91af;">ElementId</span>.InvalidElementId;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Result</span>&nbsp;Execute(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">string</span>&nbsp;message,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementSet</span>&nbsp;elements&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">try</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;initStuff(&nbsp;commandData&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;m_doc&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Failed;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;host&nbsp;=&nbsp;getRebarHost(&nbsp;commandData&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;host&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">MessageBox</span>.Show(&nbsp;<span style="color:#a31515;">&quot;Null&nbsp;host&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarShape</span>&nbsp;shape&nbsp;=&nbsp;getRebarShape();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;shape&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">MessageBox</span>.Show(&nbsp;<span style="color:#a31515;">&quot;Null&nbsp;shape&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarBarType</span>&nbsp;rebarType&nbsp;=&nbsp;getRebarBarType();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;rebarType&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">MessageBox</span>.Show(&nbsp;<span style="color:#a31515;">&quot;Null&nbsp;rebarType&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geometryElement&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;host.get_Geometry(&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Options</span>()&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;this&nbsp;will&nbsp;get&nbsp;the&nbsp;edges&nbsp;in&nbsp;family&nbsp;coordinates&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;not&nbsp;the&nbsp;global&nbsp;coordinates</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;edges&nbsp;=&nbsp;getFaceEdges(&nbsp;geometryElement&nbsp;);&nbsp;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Transform</span>&nbsp;trf&nbsp;=&nbsp;<span style="color:#2b91af;">Transform</span>.Identity;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FamilyInstance</span>&nbsp;famInst&nbsp;=&nbsp;host&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">FamilyInstance</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;famInst&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;trf&nbsp;=&nbsp;famInst.GetTransform();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;SOLUTION&nbsp;1</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;origin,&nbsp;xAxisDir,&nbsp;yAxisDir,&nbsp;xAxisBox,&nbsp;yAxisBox;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;getOriginXandYvecFromFaceEdges(&nbsp;edges,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">out</span>&nbsp;origin,&nbsp;<span style="color:blue;">out</span>&nbsp;xAxisDir,&nbsp;<span style="color:blue;">out</span>&nbsp;yAxisDir,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">out</span>&nbsp;xAxisBox,&nbsp;<span style="color:blue;">out</span>&nbsp;yAxisBox&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;we&nbsp;obtained&nbsp;origin,&nbsp;xAxis,&nbsp;yAxis&nbsp;in&nbsp;family&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;coordinates.&nbsp;Now&nbsp;we&nbsp;will&nbsp;transform&nbsp;them&nbsp;in&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;global&nbsp;coordinates</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;origin&nbsp;=&nbsp;trf.OfPoint(&nbsp;origin&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisDir&nbsp;=&nbsp;trf.OfVector(&nbsp;xAxisDir&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisDir&nbsp;=&nbsp;trf.OfVector(&nbsp;yAxisDir&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisBox&nbsp;=&nbsp;trf.OfVector(&nbsp;xAxisBox&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisBox&nbsp;=&nbsp;trf.OfVector(&nbsp;yAxisBox&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;tr&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;m_doc&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tr.Start(&nbsp;<span style="color:#a31515;">&quot;Create&nbsp;Rebar&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Rebar</span>&nbsp;createdStirrupRebar&nbsp;=&nbsp;<span style="color:#2b91af;">Rebar</span>.CreateFromRebarShape(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_doc,&nbsp;shape,&nbsp;rebarType,&nbsp;host,&nbsp;origin,&nbsp;xAxisDir,&nbsp;yAxisDir&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarShapeDrivenAccessor</span>&nbsp;rebarStirrupShapeDrivenAccessor
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;createdStirrupRebar.GetShapeDrivenAccessor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rebarStirrupShapeDrivenAccessor.SetLayoutAsFixedNumber(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5,&nbsp;10,&nbsp;<span style="color:blue;">true</span>,&nbsp;<span style="color:blue;">true</span>,&nbsp;<span style="color:blue;">true</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rebarStirrupShapeDrivenAccessor.ScaleToBox(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;origin,&nbsp;xAxisBox,&nbsp;yAxisBox&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tr.Commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;SOLUTION&nbsp;2</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;rebarSegments&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;transform&nbsp;the&nbsp;edges&nbsp;in&nbsp;global&nbsp;coordinates</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;rebarSegms&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;curve&nbsp;<span style="color:blue;">in</span>&nbsp;edges&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rebarSegms.Add(&nbsp;curve.CreateTransformed(&nbsp;trf&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Here&nbsp;you&nbsp;can&nbsp;also&nbsp;offset&nbsp;the&nbsp;curves&nbsp;to&nbsp;respect&nbsp;the&nbsp;cover.</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;tr&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;m_doc&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tr.Start(&nbsp;<span style="color:#a31515;">&quot;Create&nbsp;Rebar&nbsp;from&nbsp;Curves&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Rebar</span>&nbsp;createdStirrupRebar&nbsp;=&nbsp;<span style="color:#2b91af;">Rebar</span>.CreateFromCurves(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_doc,&nbsp;<span style="color:#2b91af;">RebarStyle</span>.StirrupTie,&nbsp;rebarType,&nbsp;<span style="color:blue;">null</span>,&nbsp;<span style="color:blue;">null</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;host,&nbsp;-<span style="color:#2b91af;">XYZ</span>.BasisX,&nbsp;rebarSegms,&nbsp;<span style="color:#2b91af;">RebarHookOrientation</span>.Left,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarHookOrientation</span>.Left,&nbsp;<span style="color:blue;">true</span>,&nbsp;<span style="color:blue;">false</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarShapeDrivenAccessor</span>&nbsp;rebarStirrupShapeDrivenAccessor
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;createdStirrupRebar.GetShapeDrivenAccessor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rebarStirrupShapeDrivenAccessor.SetLayoutAsFixedNumber(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;10,&nbsp;10,&nbsp;<span style="color:blue;">true</span>,&nbsp;<span style="color:blue;">true</span>,&nbsp;<span style="color:blue;">true</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tr.Commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">catch</span>(&nbsp;<span style="color:#2b91af;">Exception</span>&nbsp;e&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TaskDialog</span>.Show(&nbsp;<span style="color:#a31515;">&quot;exception&quot;</span>,&nbsp;e.Message&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Failed;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">void</span>&nbsp;getOriginXandYvecFromFaceEdges(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;edges,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;origin,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;xAxisDir,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;yAxisDir,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;xAxisBox,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;yAxisBox&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;origin&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisDir&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisDir&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisBox&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisBox&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;minZ&nbsp;=&nbsp;<span style="color:blue;">double</span>.MaxValue;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;ii&nbsp;=&nbsp;0;&nbsp;ii&nbsp;&lt;&nbsp;edges.Count;&nbsp;ii++&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;edgeLine&nbsp;=&nbsp;edges[ii]&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Line</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;edgeLine&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">continue</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;nextii&nbsp;=&nbsp;(&nbsp;ii&nbsp;+&nbsp;1&nbsp;)&nbsp;%&nbsp;edges.Count;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;edgeLineNext&nbsp;=&nbsp;edges[nextii]&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Line</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;edgeLineNext&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">continue</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;pntEnd&nbsp;=&nbsp;edgeLine.Evaluate(&nbsp;1,&nbsp;<span style="color:blue;">true</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;pntEnd.Z&nbsp;&lt;&nbsp;minZ&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;minZ&nbsp;=&nbsp;pntEnd.Z;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;origin&nbsp;=&nbsp;pntEnd;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;These&nbsp;two&nbsp;will&nbsp;be&nbsp;used&nbsp;by&nbsp;Rebar.CreateFromRebarShape.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;For&nbsp;this&nbsp;the&nbsp;length&nbsp;is&nbsp;irrelevant:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisDir&nbsp;=&nbsp;edgeLine.Direction&nbsp;*&nbsp;-1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisDir&nbsp;=&nbsp;edgeLineNext.Direction;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;These&nbsp;two&nbsp;&nbsp;will&nbsp;be&nbsp;used&nbsp;at&nbsp;Rebar.ScaleToBox.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;For&nbsp;this,&nbsp;the&nbsp;length&nbsp;is&nbsp;important,&nbsp;because&nbsp;it</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;will&nbsp;represent&nbsp;the&nbsp;length&nbsp;of&nbsp;box&nbsp;segment.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xAxisBox&nbsp;=&nbsp;xAxisDir&nbsp;*&nbsp;edgeLine.ApproximateLength;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yAxisBox&nbsp;=&nbsp;yAxisDir&nbsp;*&nbsp;edgeLineNext.ApproximateLength;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Here&nbsp;you&nbsp;can&nbsp;also&nbsp;remove&nbsp;from&nbsp;the&nbsp;length&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;the&nbsp;value&nbsp;of&nbsp;the&nbsp;cover.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;getFaceEdges(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geometryElement&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">GeometryObject</span>&nbsp;geometryObject&nbsp;<span style="color:blue;">in</span>&nbsp;geometryElement&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solid&nbsp;=&nbsp;geometryObject&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Solid</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;solid&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FaceArray</span>&nbsp;faces&nbsp;=&nbsp;solid.Faces;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Face</span>&nbsp;face&nbsp;<span style="color:blue;">in</span>&nbsp;faces&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;plane&nbsp;=&nbsp;face.GetSurface()&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Plane</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;AreEqual(&nbsp;plane.Normal.DotProduct(&nbsp;<span style="color:#2b91af;">XYZ</span>.BasisX&nbsp;),&nbsp;-1&nbsp;)&nbsp;)&nbsp;<span style="color:green;">//&nbsp;vecs&nbsp;are&nbsp;parallel&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;There&nbsp;should&nbsp;be&nbsp;onlu&nbsp;one&nbsp;curve&nbsp;loop.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;It&nbsp;can&nbsp;be&nbsp;multiple&nbsp;if&nbsp;the&nbsp;face&nbsp;have&nbsp;a&nbsp;hole.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;face.GetEdgesAsCurveLoops().Count&nbsp;!=&nbsp;1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">null</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;edgesArr&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">CurveLoopIterator</span>&nbsp;cli&nbsp;=&nbsp;face
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GetEdgesAsCurveLoops()[0]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GetCurveLoopIterator();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>(&nbsp;cli.MoveNext()&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;edge&nbsp;=&nbsp;cli.Current;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edgesArr.Add(&nbsp;edge&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;edgesArr;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryInstance</span>&nbsp;geometryInstance&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;geometryObject&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">GeometryInstance</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;geometryInstance&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;getFaceEdges(&nbsp;geometryInstance
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GetSymbolGeometry()&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;getRebarHost(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_uiApp&nbsp;=&nbsp;commandData.Application;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Selection</span>&nbsp;sel&nbsp;=&nbsp;m_uiApp.ActiveUIDocument.Selection;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Reference</span>&nbsp;refr&nbsp;=&nbsp;sel.PickObject(&nbsp;<span style="color:#2b91af;">ObjectType</span>.Element&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;m_doc.GetElement(&nbsp;refr.ElementId&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">RebarBarType</span>&nbsp;getRebarBarType()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;fec&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;m_doc&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">RebarBarType</span>&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;fec.FirstElement()&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">RebarBarType</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">RebarShape</span>&nbsp;getRebarShape()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;fec
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;m_doc&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">RebarShape</span>&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;strName&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Bügel&nbsp;Geschlossen&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">Element</span>&gt;&nbsp;shapeElems&nbsp;=&nbsp;fec.ToElements();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;shapeElem&nbsp;<span style="color:blue;">in</span>&nbsp;shapeElems&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">RebarShape</span>&nbsp;shape&nbsp;=&nbsp;shapeElem&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">RebarShape</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;shape.Name.Contains(&nbsp;strName&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;shape;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;AreEqual(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;firstValue,&nbsp;<span style="color:blue;">double</span>&nbsp;secondValue,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;tolerance&nbsp;=&nbsp;1.0e-9&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(&nbsp;secondValue&nbsp;-&nbsp;tolerance&nbsp;&lt;&nbsp;firstValue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&amp;&amp;&nbsp;firstValue&nbsp;&lt;&nbsp;secondValue&nbsp;+&nbsp;tolerance&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">void</span>&nbsp;initStuff(&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_uiApp&nbsp;=&nbsp;commandData.Application;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m_doc&nbsp;=&nbsp;m_uiApp.ActiveUIDocument.Document;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}
</pre>

**Response:** Yes, your reply was helpful, Thanks. It has proven that the cause is indeed the vectors.

This is also related to, and actually the cause and solution for another issue that we encountered 
with [missing and zero length long rebars](https://forums.autodesk.com/t5/revit-api-forum/missing-zero-length-long-rebars/m-p/8293340).

The main conclusion is that the vectors are really very, very, very, sensitive.

Here is example:

Obtained from face as suggested in your example:

<pre>
  xAxisDir: (-0,4226182617, 0,9063077870, 0,0000000000)
  yAxisDir: (-0,9063077870, -0,4226182617, 0,0000000000)
</pre>

My previous vectors (only normalized to unit):

<pre>
  xVec unit: (-0,4226177303, 0,9063080349, 0,0000000000)
  yVec unit: (-0,9063077319, -0,4226183800, 0,0000000000)
</pre>

You can see that these vectors differ generally on some 6th decimal place. Is this really convenient?

I think it would be more comfortable if your API method would be intelligent or adaptive to correct values for appropriate layout/graphics.

Who cares about one-millionth in unit vector?

I retrieve the vectors based on intersection with plane in the middle of the beam. Face of the beam could had been modified according to connected beam/column (and also comparing them to get to lower left corner) &ndash; so this retrieval is a bit complicated.

**Answer:** Thank you for confirming and sorry the API is so fussy. That is normal, I'm afraid, both in Revit and elsewhere...  :-)

The safest is to always compute all the required vectors yourself in a final step, based on your required input, then normalised and cross-product calculated to really be rock solid and precise.

Many thanks to Stefan Dobre, Autodesk Principal Engineer in Romania, for his effective research, solution and documentation of this issue!
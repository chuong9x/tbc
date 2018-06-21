<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<!--
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---

- https://twitter.com/jeremytammik/status/1009750141748875264
  Forge for Digital Construction at #bamdigital with Az Jasat and @Manu_Venugopal, presenting on Insight and Connection #RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon

- 14337750 [Invoke the "Draw Model Line" Command]

Digital construction slide deck and drawing a model line in the #RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon 

I am attending the BAM <i>Digital Construction Live</i> event in the UK and presenting on Forge for that domain.
Today, I'll share my slide deck from this event and welcome my colleague Xiaodong answering his first Revit API cases
&ndash; Forge for Digital Construction
&ndash; Welcome Xiaodong and invoking the <i>Draw Model Line</i> command...

--->

### Digital Construction and Drawing a Model Line

I am attending the BAM <i>Digital Construction Live</i> event in the UK and presenting on Forge for that domain.
You can see what's going on here looking for
the [#bamdigital](https://twitter.com/search?q=%23bamdigital) hashtag.

On the way here, I visited my brother and passed by the interesting climbing areas
at [Cheddar Gorge](https://en.wikipedia.org/wiki/Cheddar_Gorge)
and [Symonds Yat](https://en.wikipedia.org/wiki/Symonds_Yat):

<center>
<img src="img/119_jeremy_leading_600.jpg" alt="Cheddar Gorge" width="200"/>
</center>

In the latter, we climbed the Long Rock Pinnacle via Whitt:

<center>
<img src="img/143_jeremy_long_rock_pinnacle_800x600.jpg" alt="Long Rock Pinnacle" width="400"/>
</center>

Today, I'll share my slide deck from this event and welcome my colleague Xiaodong answering his first Revit API cases:

- [Forge for Digital Construction](#2) 
- [Welcome Xiaodong and invoking the *Draw Model Line* command](#3) 


#### <a name="2"></a> Forge for Digital Construction

As said, I am attending this event with my Autodesk colleagues Az Jasat and Manu Venugopal, presenting on Insight and Connection.

Insight is covered by Manu, discussing Project IQ and machine learning enhanced analytics for automated risk assessment.

I am presenting on Forge for the digital construction process and connecting to the BIM360 products.

I already explained the main concepts from my point of view in 
the [overview of Forge for AEC and BIM360](http://thebuildingcoder.typepad.com/blog/2018/06/forge-for-aec-and-bim360-overview.html) and
the [BIM360 and Forge for AEC message and samples](http://thebuildingcoder.typepad.com/blog/2018/06/bim360-and-forge-for-aec-real-message-and-samples.html).

Here is the final slide deck summarising those points in
the [BAM Forge for Digital Construction slides](zip/bam_bim360_forge_aec_slides.pdf).

<center>
<img src="img/187_manu_and_az_722x380.jpg" alt="Manu and Az at BAM Digital Construction Live" width="722"/>
</center>


#### <a name="3"></a> Welcome Xiaodong and Invoking the *Draw Model Line* Command

Many congratulations to my colleague Xiaodong Liang for diving into the Revit API and starting to answer cases!

Here is one of his first:

**Question:** How can I programmatically invoke the *Draw Model Line* command?

**Answer:** If your workflow is to simply to invoke the built-in *Draw Model Line* command as is, you could find the command id and execute it using `PostCommand`:

<pre class="code">
&nbsp;&nbsp;Autodesk.Revit.UI.<span style="color:#2b91af;">RevitCommandId</span>&nbsp;cmd_id
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:#2b91af;">RevitCommandId</span>.LookupPostableCommandId(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">PostableCommand</span>.ModelLine&nbsp;);
 
&nbsp;&nbsp;uiapp.PostCommand(&nbsp;cmd_id&nbsp;);
</pre>

If your workflow is to create a model line yourself with the parameters the user inputs, you can use the following:

<pre class="code">
  <span style="color:#2b91af;">UIApplication</span>&nbsp;uiApp&nbsp;=&nbsp;commandData.Application;
  <span style="color:#2b91af;">Application</span>&nbsp;rvtApp&nbsp;=&nbsp;uiApp.Application;
  <span style="color:#2b91af;">UIDocument</span>&nbsp;uiDoc&nbsp;=&nbsp;uiApp.ActiveUIDocument;
  <span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;=&nbsp;uiDoc.Document;
   
  <span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;transaction&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;doc&nbsp;)&nbsp;)
  {
  &nbsp;&nbsp;transaction.Start(&nbsp;<span style="color:#a31515;">&quot;Create&nbsp;Model&nbsp;Line&nbsp;By&nbsp;Me&quot;</span>&nbsp;);
   
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;startPoint&nbsp;=&nbsp;<span style="color:#2b91af;">XYZ</span>.Zero;
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;endPoint&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>(&nbsp;10,&nbsp;10,&nbsp;0&nbsp;);
   
  &nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;geomLine&nbsp;=&nbsp;<span style="color:#2b91af;">Line</span>.CreateBound(&nbsp;startPoint,&nbsp;endPoint&nbsp;);
   
  &nbsp;&nbsp;<span style="color:green;">//&nbsp;Create&nbsp;a&nbsp;geometry&nbsp;plane&nbsp;in&nbsp;Revit&nbsp;application&nbsp;memory</span>
   
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;origin&nbsp;=&nbsp;<span style="color:#2b91af;">XYZ</span>.Zero;
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;normal&nbsp;=&nbsp;<span style="color:#2b91af;">XYZ</span>.BasisZ;
  &nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;geomPlane&nbsp;=&nbsp;<span style="color:#2b91af;">Plane</span>.CreateByNormalAndOrigin(
  &nbsp;&nbsp;&nbsp;&nbsp;normal,&nbsp;origin&nbsp;);
   
  &nbsp;&nbsp;<span style="color:green;">//&nbsp;Create&nbsp;a&nbsp;sketch&nbsp;plane&nbsp;in&nbsp;current&nbsp;document</span>
   
  &nbsp;&nbsp;<span style="color:#2b91af;">SketchPlane</span>&nbsp;sketch&nbsp;=&nbsp;<span style="color:#2b91af;">SketchPlane</span>.Create(
  &nbsp;&nbsp;&nbsp;&nbsp;doc,&nbsp;geomPlane&nbsp;);
   
  &nbsp;&nbsp;<span style="color:#2b91af;">ModelLine</span>&nbsp;modelLine&nbsp;=&nbsp;doc.Create.NewModelCurve(
  &nbsp;&nbsp;&nbsp;&nbsp;geomLine,&nbsp;sketch&nbsp;)&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">ModelLine</span>;
   
  &nbsp;&nbsp;transaction.Commit();
  }
</pre>

Xiaodong adds:

> Though it is a simple question, it took me some time to test out the working codes, and I learned some valuable knowledge of Revit API.

> Stumblingly at the starting point of the journey into the Revit API &nbsp; :-)

Many thanks to Xiaodong for the nice and comprehensive answer!
 
It looks like a great start!
 
I wish you much success and lots of fun going further.

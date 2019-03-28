<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---


twitter:

 in the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon 

We have looked at numerous different approaches to determine wall openings in the past, so it seems pretty hard to nail down, and pretty important to solve.
Now Håvard Leding of Symetri contributed yet another exciting idea which highlights a number of surprising aspects,
demonstrates a further creative use case for <code>GetDependentElements<code> and expands on his 
recent RevitLookup enhancement to retrieve and snoop dependent elements
&ndash; Get demolished solid
&ndash; Why?
&ndash; Questions...

linkedin:


of [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.145.4).

-->

### Determine Exact Opening by Demolishing

Here comes the most surprising Revit API functionality I have ever seen, put to a very useful and common task, determining the exact wall opening required for a door or window.

We have looked at numerous different approaches to determine wall openings in the past, including:

- [Opening geometry](http://thebuildingcoder.typepad.com/blog/2012/01/opening-geometry.html)
- [The temporary transaction trick for gross slab data](http://thebuildingcoder.typepad.com/blog/2012/10/the-temporary-transaction-trick-for-gross-slab-data.html)
- [Retrieving wall openings and sorting points](http://thebuildingcoder.typepad.com/blog/2015/12/retrieving-wall-openings-and-sorting-points.html)
- [Wall opening profiles](http://thebuildingcoder.typepad.com/blog/2015/12/wall-opening-profiles-and-happy-holidays.html#3)
- [Determining wall opening areas per room](http://thebuildingcoder.typepad.com/blog/2016/04/determining-wall-opening-areas-per-room.html#4)
- [More on wall opening areas per room](http://thebuildingcoder.typepad.com/blog/2016/04/more-on-wall-opening-areas-per-room.html)
- [Two energy model types](http://thebuildingcoder.typepad.com/blog/2017/01/family-category-and-two-energy-model-types.html#3)
- [IFC helper returns outer `CurveLoop` of door or window](https://thebuildingcoder.typepad.com/blog/2017/06/copy-local-false-and-ifc-utils-for-wall-openings.html#2)

So, it seems pretty hard to nail down, and pretty important to solve.

Now Håvard Leding of [Symetri](https://www.symetri.com) contributed
yet another exciting idea which highlights a number of surprising aspects,
demonstrates a further creative use case for `GetDependentElements` and expands on his 
recent [RevitLookup enhancement to retrieve and snoop dependent elements](https://thebuildingcoder.typepad.com/blog/2019/03/retrieving-and-snooping-dependent-elements.html):

- [Get Demolished Solid](#2) 
- [Why?](#3) 
- [Questions?](#4) 

In his own words:


#### <a name="2"></a> Get Demolished Solid

Here is another use case for `GetDependentElements()`.
 
Determining the opening dimensions for Doors and Windows is surprisingly difficult, since you can't trust their parameters or reference planes to be consistent.

I suggest this alternative method that uses the solid that Revit creates when you demolish an opening.

It also uses the good
old [temporary transaction trick](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53):

<pre class="code">
<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;If&nbsp;you&nbsp;demolish&nbsp;a&nbsp;door,&nbsp;Revit&nbsp;will&nbsp;automatically&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;fill&nbsp;the&nbsp;opening&nbsp;with&nbsp;a&nbsp;wall.&nbsp;So,&nbsp;we&nbsp;will&nbsp;use&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;that&nbsp;wall&nbsp;to&nbsp;get&nbsp;opening&nbsp;dimensions.</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">param</span><span style="color:gray;">&nbsp;name</span><span style="color:gray;">=</span><span style="color:gray;">&quot;</span>fi<span style="color:gray;">&quot;</span><span style="color:gray;">&gt;</span><span style="color:green;">Door&nbsp;or&nbsp;Window&nbsp;is&nbsp;expected</span><span style="color:gray;">&lt;/</span><span style="color:gray;">param</span><span style="color:gray;">&gt;</span>
<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;GetDemolishedSolid(&nbsp;
&nbsp;&nbsp;<span style="color:#2b91af;">FamilyInstance</span>&nbsp;fi&nbsp;)
{
&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;=&nbsp;fi.Document;
&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solidOpening&nbsp;=&nbsp;<span style="color:blue;">null</span>;
 
&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;t&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;doc&nbsp;)&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;fi.HasPhases()&nbsp;&amp;&amp;&nbsp;fi.ArePhasesModifiable()&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t.Start(&nbsp;<span style="color:#a31515;">&quot;temp&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fi.DemolishedPhaseId&nbsp;=&nbsp;fi.CreatedPhaseId;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;doc.Regenerate();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IList&lt;<span style="color:#2b91af;">ElementId</span>&gt;&nbsp;dependents&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;fi.GetDependentElements(&nbsp;<span style="color:blue;">null</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;id&nbsp;<span style="color:blue;">in</span>&nbsp;dependents&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;e&nbsp;=&nbsp;doc.GetElement(&nbsp;id&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;e&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:#2b91af;">Wall</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geomWall&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;e.get_Geometry(&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Options</span>()&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Need&nbsp;to&nbsp;clone&nbsp;it,&nbsp;as&nbsp;Rollback&nbsp;will&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;destroy&nbsp;the&nbsp;original&nbsp;solid.</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;solidOpening&nbsp;=&nbsp;<span style="color:#2b91af;">SolidUtils</span>.Clone(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span style="color:#2b91af;">Solid</span>)&nbsp;geomWall.First()&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t.RollBack();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
 
&nbsp;&nbsp;<span style="color:green;">//&nbsp;In&nbsp;the&nbsp;family&nbsp;editor,&nbsp;the&nbsp;host&nbsp;wall&nbsp;is&nbsp;always&nbsp;</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;aligned&nbsp;ortho&nbsp;to&nbsp;the&nbsp;family&nbsp;coordinate&nbsp;system,</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;so&nbsp;we&nbsp;can&nbsp;get&nbsp;the&nbsp;opening&nbsp;dimensions&nbsp;directly&nbsp;</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;from&nbsp;the&nbsp;bounding&nbsp;box.</span>
 
&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solidInFamilyCoordinates&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:#2b91af;">SolidUtils</span>.CreateTransformed(&nbsp;solidOpening,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;fi.GetTotalTransform().Inverse&nbsp;);
 
&nbsp;&nbsp;<span style="color:#2b91af;">BoundingBoxXYZ</span>&nbsp;bb&nbsp;=&nbsp;solidInFamilyCoordinates.GetBoundingBox();
 
&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;dimensions&nbsp;=&nbsp;bb.Max&nbsp;-&nbsp;bb.Min;
 
&nbsp;&nbsp;<span style="color:#2b91af;">TaskDialog</span>.Show(&nbsp;<span style="color:#a31515;">&quot;Opening&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;Opening&nbsp;is&nbsp;&quot;</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;dimensions.X.ToString()&nbsp;+&nbsp;<span style="color:#a31515;">&quot;&nbsp;by&nbsp;&quot;</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;dimensions.Z.ToString()&nbsp;);
 
&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;solidOpening;
}
</pre>
 

#### <a name="3"></a> Why?

I agree some more explanation as to why parameters and refplanes are not to be trusted would be nice.
 
You can't rely on any built-in parameters being in use.
Or used as you expect them to be used.
Same goes for reference planes.
 
Sometimes, it's just static geometry. (No parameters in use)
And sometimes, it's just an opening family. (No solids)
 
Still, you need to analyse something, and preferably solids.
But sometimes, solids don't represent the true opening, like in this case, where you have a gap:
 
<center>
<img src="img/window_opening_with_gap.jpg" alt="Window opening with gap" width="302">
</center>
 
It's just safer and a lot easier to use the final 'opening solid' from a demolished state.
 
If you want to try it out yourself, the method is easy enough to use; 
it just requires a door or window to run &nbsp; :-)
 

#### <a name="4"></a> Questions?

**Question:** Reading the code in more detail, though, I don't really understand...
 
The family instance `fi` is a window, for example, yes?
 
From the window, you call `GetDependentElements`, which returns "all elements that, from a logical point of view, are the children of this Element".
 
From those, you extract the wall.
 
Hmm. So, the wall is dependent on the window? That seems weird.
 
OK, let's accept that the wall is returned.
 
Now, from the wall, you retrieve the first geometry element.
 
That is now saved as the solid opening.
 
That seems super weird. I would have thought that the wall geometry is the wall geometry, and that the hole is a hole.
 
Why is the first solid in the wall geometry a solid representing the opening?
 
Does that really work?
 
Can you explain?
 
**Answer:** Yes, it works; just try it :-)
 
It works, because when you demolish openings, Revit automatically fills the entire hole with a new Wall.
You can see it if you manually demolish a window.
There is no longer a hole in the wall.
 
Very easy to get dimensions from the bounding box, once transformed into the family editor coordinate system.
Because there, the host wall direction is always aligned with the coordinate system cardinal axes.
I guess opening dimensions could be extracted without the transform, but I wouldn't know how to do that.
 
You could also analyse the shape of the opening by looking at the vertical front face of the solid.

You would have to do so to handle cases where the window is not rectangular, e.g., circular or something else.

**Response:** Wow. I am impressed. And surprised.

Many thanks to Håvard for this innovative solution and interesting explanation!

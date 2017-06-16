<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="run_prettify.js" type="text/javascript"></script>
<!--
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---

- dimension segment endpoints
  13073404 [How to retrieve a Dimension's (segment) geometry ?] 
  https://forums.autodesk.com/t5/revit-api-forum/how-to-retrieve-a-dimension-s-segment-geometry/m-p/7145688
  https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdGetDimensionPoints.cs

Determining Dimension Segment Endoints #RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon 

A rather hard struggle led to a rather simple solution for determining the start and end points of dimension segments. In summary, the solution looks like this &ndash; A <code>Dimension</code> element is either single- or multi-segment; these two cases need to be handled separately &ndash; In case of a single segment, the dimension element itself has a line, an origin and a value; the line is indeed the dimension line. However, it may be unbounded...

-->

### Determining Dimension Segment Endoints

A rather hard struggle led to a rather simple solution for determining the start and end points of dimension segments.

In summary, the solution looks like this:

- A `Dimension` element is either single- or multi-segment; these two cases need to be handled separately.
- In case of a single segment, the dimension element itself has a line, an origin and a value; the line is indeed the dimension line. However, it may be unbounded, so you cannot determine the start or end point from it. It does provide a direction, though, and the origin is in the middle, so you can determine the start and end points by going forward and backward along the line by half the length:

<pre>
  direction = line.direction.normal;
  start = origin - half * length * direction;
  end = origin + half * length * direction;
</pre>

- In the case of multiple segments, use an analogue approach on each individual segment.

I implemented and added a new external
command [CmdGetDimensionPoints](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdGetDimensionPoints.cs)
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
I added it 
to [The Building Coder samples](https://github.com/jeremytammik/the_building_coder_samples)
[release 2018.0.134.0](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.134.0) that
calculates the segment endpoints accordingly.

Here is a dimension between three parallel walls:

<center>
<img src="img/dim_three_walls_final.png" alt="Dimensioning of three walls" width="300"> 
</center>

`CmdGetDimensionPoints` adds the green model line markers denoting the dimension origin in the middle of the first segment and at the three start and end points of the two segments calculated along the dimension line from that starting point:

<center>
<img src="img/dim_three_walls_markers_final.png" alt="Dimension origin and segment points" width="300"> 
</center>

For the full research and discussion leading up to this, please refer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread 
on [how to retrieve a dimension's segment  geometry](https://forums.autodesk.com/t5/revit-api-forum/how-to-retrieve-a-dimension-s-segment-geometry/m-p/7145688).

Many thanks to Fair59 for his important contributions and to Jonathan 'Maisoui' for raising the issue.

Here is the full final implementation based on this:

<pre class="code">
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Return&nbsp;dimension&nbsp;origin,&nbsp;i.e.,&nbsp;the&nbsp;midpoint</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;of&nbsp;the&nbsp;dimension&nbsp;or&nbsp;of&nbsp;its&nbsp;first&nbsp;segment.</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;GetDimensionStartPoint(
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Dimension</span>&nbsp;dim&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p&nbsp;=&nbsp;<span style="color:blue;">null</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">try</span>
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;dim.Origin;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">catch</span>(&nbsp;Autodesk.Revit.Exceptions.<span style="color:#2b91af;">ApplicationException</span>&nbsp;ex&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;ex.Message.Equals(&nbsp;<span style="color:#a31515;">&quot;Cannot&nbsp;access&nbsp;this&nbsp;method&nbsp;if&nbsp;this&nbsp;dimension&nbsp;has&nbsp;more&nbsp;than&nbsp;one&nbsp;segment.&quot;</span>&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">DimensionSegment</span>&nbsp;seg&nbsp;<span style="color:blue;">in</span>&nbsp;dim.Segments&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;seg.Origin;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;p;
&nbsp;&nbsp;}
 
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Retrieve&nbsp;the&nbsp;start&nbsp;and&nbsp;end&nbsp;points&nbsp;of</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;each&nbsp;dimension&nbsp;segment,&nbsp;based&nbsp;on&nbsp;the&nbsp;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;dimension&nbsp;origin&nbsp;determined&nbsp;above.</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;&nbsp;GetDimensionPoints(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Dimension</span>&nbsp;dim,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;pStart&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;dimLine&nbsp;=&nbsp;dim.Curve&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Line</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;dimLine&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;&nbsp;pts&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;dimLine.MakeBound(&nbsp;0,&nbsp;1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;pt1&nbsp;=&nbsp;dimLine.GetEndPoint(&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;pt2&nbsp;=&nbsp;dimLine.GetEndPoint(&nbsp;1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;direction&nbsp;=&nbsp;pt2.Subtract(&nbsp;pt1&nbsp;).Normalize();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;0&nbsp;==&nbsp;dim.Segments.Size&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;v&nbsp;=&nbsp;0.5&nbsp;*&nbsp;(<span style="color:blue;">double</span>)dim.Value&nbsp;*&nbsp;direction;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pts.Add(&nbsp;pStart&nbsp;-&nbsp;v&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pts.Add(&nbsp;pStart&nbsp;+&nbsp;v&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p&nbsp;=&nbsp;pStart;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">DimensionSegment</span>&nbsp;seg&nbsp;<span style="color:blue;">in</span>&nbsp;dim.Segments&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;v&nbsp;=&nbsp;(<span style="color:blue;">double</span>)&nbsp;seg.Value&nbsp;*&nbsp;direction;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;0&nbsp;==&nbsp;pts.Count)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pts.Add(&nbsp;p&nbsp;=&nbsp;(&nbsp;pStart&nbsp;-&nbsp;0.5&nbsp;*&nbsp;v&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pts.Add(&nbsp;p&nbsp;=&nbsp;p.Add(&nbsp;v&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;pts;
&nbsp;&nbsp;}
 
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Graphical&nbsp;debugging&nbsp;helper&nbsp;using&nbsp;model&nbsp;lines</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;to&nbsp;draw&nbsp;an&nbsp;X&nbsp;at&nbsp;the&nbsp;given&nbsp;position.</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:blue;">void</span>&nbsp;DrawMarker(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;size,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SketchPlane</span>&nbsp;sketchPlane&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;size&nbsp;*=&nbsp;0.5;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;v&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>(&nbsp;size,&nbsp;size,&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;=&nbsp;sketchPlane.Document;
&nbsp;&nbsp;&nbsp;&nbsp;doc.Create.NewModelCurve(&nbsp;<span style="color:#2b91af;">Line</span>.CreateBound(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;-&nbsp;v,&nbsp;p&nbsp;+&nbsp;v&nbsp;),&nbsp;sketchPlane&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;v&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">XYZ</span>(&nbsp;size,&nbsp;-size,&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;doc.Create.NewModelCurve(&nbsp;<span style="color:#2b91af;">Line</span>.CreateBound(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;-&nbsp;v,&nbsp;p&nbsp;+&nbsp;v&nbsp;),&nbsp;sketchPlane&nbsp;);
&nbsp;&nbsp;}
 
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Result</span>&nbsp;Execute(
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">string</span>&nbsp;message,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementSet</span>&nbsp;elements&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UIApplication</span>&nbsp;uiapp&nbsp;=&nbsp;commandData.Application;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UIDocument</span>&nbsp;uidoc&nbsp;=&nbsp;uiapp.ActiveUIDocument;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;=&nbsp;uidoc.Document;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Selection</span>&nbsp;sel&nbsp;=&nbsp;uidoc.Selection;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ISelectionFilter</span>&nbsp;f
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JtElementsOfClassSelectionFilter</span>&lt;<span style="color:#2b91af;">Dimension</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Reference</span>&nbsp;elemRef&nbsp;=&nbsp;sel.PickObject(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ObjectType</span>.Element,&nbsp;f,&nbsp;<span style="color:#a31515;">&quot;Pick&nbsp;a&nbsp;dimension&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Dimension</span>&nbsp;dim&nbsp;=&nbsp;doc.GetElement(&nbsp;elemRef&nbsp;)&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Dimension</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p&nbsp;=&nbsp;GetDimensionStartPoint(&nbsp;dim&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;&nbsp;pts&nbsp;=&nbsp;GetDimensionPoints(&nbsp;dim,&nbsp;p&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;n&nbsp;=&nbsp;pts.Count;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;<span style="color:#a31515;">&quot;Dimension&nbsp;origin&nbsp;at&nbsp;{0}&nbsp;followed&nbsp;&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:#a31515;">&quot;by&nbsp;{1}&nbsp;further&nbsp;point{2}{3}&nbsp;{4}&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Util</span>.PointString(&nbsp;p&nbsp;),&nbsp;n,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Util</span>.PluralSuffix(&nbsp;n&nbsp;),&nbsp;<span style="color:#2b91af;">Util</span>.DotOrColon(&nbsp;n&nbsp;),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>.Join(&nbsp;<span style="color:#a31515;">&quot;,&nbsp;&quot;</span>,&nbsp;pts.Select(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;q&nbsp;=&gt;&nbsp;<span style="color:#2b91af;">Util</span>.PointString(&nbsp;q&nbsp;)&nbsp;)&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">double</span>&gt;&nbsp;d&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">double</span>&gt;(&nbsp;n&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;q0&nbsp;=&nbsp;p;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;q&nbsp;<span style="color:blue;">in</span>&nbsp;pts&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d.Add(&nbsp;q.X&nbsp;-&nbsp;q0.X&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;q0&nbsp;=&nbsp;q;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;Horizontal&nbsp;distances&nbsp;in&nbsp;metres:&nbsp;&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:blue;">string</span>.Join(&nbsp;<span style="color:#a31515;">&quot;,&nbsp;&quot;</span>,&nbsp;d.Select(&nbsp;x&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Util</span>.RealString(&nbsp;<span style="color:#2b91af;">Util</span>.FootToMetre(&nbsp;x&nbsp;)&nbsp;)&nbsp;)&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;tx&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;doc&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.Start(&nbsp;<span style="color:#a31515;">&quot;Draw&nbsp;Point&nbsp;Markers&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SketchPlane</span>&nbsp;sketchPlane&nbsp;=&nbsp;dim.View.SketchPlane;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;size&nbsp;=&nbsp;0.3;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawMarker(&nbsp;p,&nbsp;size,&nbsp;sketchPlane&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pts.ForEach(&nbsp;q&nbsp;=&gt;&nbsp;DrawMarker(&nbsp;q,&nbsp;size,&nbsp;sketchPlane&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.Commit();
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;}
</pre>


<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<!--
<script src="run_prettify.js" type="text/javascript"></script>
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- you cannot rotate the outline; rotate the elements instead, or create a solid yourself
  https://forums.autodesk.com/t5/revit-api-forum/boundingbox-outline-and-boundingboxisinsidefilter/m-p/7921336
  
#RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon

This is pretty obvious, once you think about it, and apparently worth pointing out anyway:
The outline defining a bounding box filter is always aligned with the cardinal axes
&ndash; Rotating `Min` and `Max` distorts the box
&ndash; Rotate target elements or use a solid filter...

--->

### Bounding Box Filter is Always Axis Aligned

This is pretty obvious, once you think about it, and apparently worth pointing out anyway:

The outline defining a bounding box filter is always aligned with the cardinal axes.

This question was clarified in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread 
on [`BoundingBox` outline and `BoundingBoxIsInsideFilter`](https://forums.autodesk.com/t5/revit-api-forum/boundingbox-outline-and-boundingboxisinsidefilter/m-p/7921336):

- [Rotating `Min` and `Max` distorts the box](#2) 
- [Rotate target elements or use a solid filter](#3) 

####<a name="2"></a>Rotating Min and Max Distorts the Box

**Question:** I have a question about a bounding box.

I tried to get its outline and retrieve the elements inside it.

I encountered 3 cases:

- Rotation = 0 deg &ndash; My code works fine with a horizontal bounding box:

<center>
<img src="img/rotated_bounding_box_1.png" alt="Bounding box rotated 0 degrees" width="545"/>
</center>

- Rotation = 10 ~ 20 deg &ndash; I can still get the BoundingBox outline, but cannot select all elements inside:

<center>
<img src="img/rotated_bounding_box_2.png" alt="Bounding box rotated 10 ~ 20 degrees" width="575"/>
</center>

- Rotation > 45 deg &ndash; I cannot get the BoundingBox outline. The error message says, "outline is an empty outline":

<center>
<img src="img/rotated_bounding_box_3.png" alt="Bounding box rotated &gt; 45 degrees" width="621"/>
</center>

Here is the code; the exception is thrown by the `Outline` constructor:

<pre class="code">
  <span style="color:#2b91af;">View3D</span>&nbsp;curView3d&nbsp;=&nbsp;doc.ActiveView&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">View3D</span>;
  <span style="color:#2b91af;">BoundingBoxXYZ</span>&nbsp;box&nbsp;=&nbsp;curView3d.GetSectionBox();
  <span style="color:#2b91af;">Transform</span>&nbsp;t&nbsp;=&nbsp;box.Transform;
   
  <span style="color:#2b91af;">Outline</span>&nbsp;o&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Outline</span>(&nbsp;
  &nbsp;&nbsp;t.OfPoint(&nbsp;box.Min&nbsp;),&nbsp;
  &nbsp;&nbsp;t.OfPoint(&nbsp;box.Max&nbsp;)&nbsp;);
   
  <span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;collector&nbsp;
  &nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;doc&nbsp;);
   
  <span style="color:#2b91af;">BoundingBoxIsInsideFilter</span>&nbsp;bbfilter&nbsp;
  &nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">BoundingBoxIsInsideFilter</span>(&nbsp;o&nbsp;);
   
  <span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">ElementId</span>&gt;&nbsp;insideList&nbsp;=&nbsp;collector
  &nbsp;&nbsp;.WhereElementIsNotElementType()
  &nbsp;&nbsp;.WherePasses(&nbsp;bbfilter&nbsp;)
  &nbsp;&nbsp;.Where(&nbsp;x&nbsp;=&gt;&nbsp;x.GetTypeId().IntegerValue&nbsp;!=&nbsp;-1&nbsp;)
  &nbsp;&nbsp;.Where(&nbsp;x&nbsp;=&gt;&nbsp;x.IsPhysicalElement()&nbsp;)
  &nbsp;&nbsp;.Select(&nbsp;x&nbsp;=&gt;&nbsp;x.Id&nbsp;)
  &nbsp;&nbsp;.ToList();
   
  uidoc.Selection.SetElementIds(&nbsp;insideList&nbsp;);
</pre>


####<a name="3"></a>Rotate Target Elements or Use a Solid Filter

**Answer:** As said, this is pretty obvious, once you think about it.

The outline defining a bounding box filter is always aligned with the cardinal axes.

If you simply grab the `Min` and `Max` point of the bounding box and rotate them far enough, they will end up in positions that specify an empty `Outline`.

Hence the exception.

You seem to be expecting a rotated box. Instead, it creates an axis aligned outline, which won't have the same proportions as the original box (the min and max were rotated).

You can solve this problem by doing the opposite, i.e., rotating the elements' outlines and not the BBox's outline (new outline from all rotated corners, so you lose some precision).

We ended up doing that is one case and it worked pretty well. 

Another approach that comes to mind:

Instead of using the [`BoundingBoxIsInsideFilter` class](http://www.revitapidocs.com/2018.1/eb8735d7-28fc-379d-9de9-1e02326851f5.htm) with an axis-aligned bounding box,
you could simply create your own `Solid` representing the rotated box and use
an [`ElementIntersectsSolidFilter`](http://www.revitapidocs.com/2018.1/19276b94-fa39-64bb-bfb8-c16967c83485.htm).

Note that the `BoundingBoxIsInsideFilter` is a quick filter, whereas the `ElementIntersectsSolidFilter` is slow,
cf. [Quick, Slow and LINQ Element Filtering](http://thebuildingcoder.typepad.com/blog/2015/12/quick-slow-and-linq-element-filtering.html).


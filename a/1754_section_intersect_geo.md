<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- get intersection lines from cut in section view
  https://forums.autodesk.com/t5/revit-api-forum/how-to-receive-intersection-of-section-and-familyinstance/m-p/8802202

twitter:

Retrieving section view intersection cut geometry in the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon http://bit.ly/sectioncutgeo

I played around creating a new SectionCutGeo add-in to retrieve the geometry resulting from cutting a family instance in a section view.
This was prompted the Revit API discussion forum thread on getting intersection lines from cut in section view
&ndash; Intersection of section and family instance task
&ndash; Solution options, geometry elements and view settings
&ndash; Retrieving curves in the cut plane
&ndash; Helper methods and external command mainline
&ndash; Sample model and results
&ndash; Caveat...

linkedin:

Retrieving section view intersection cut geometry in the #RevitAPI 

http://bit.ly/sectioncutgeo

I played around creating a new SectionCutGeo add-in to retrieve the geometry resulting from cutting a family instance in a section view.

This was prompted the Revit API discussion forum thread on getting intersection lines from cut in section view:

- Intersection of section and family instance task
- Solution options, geometry elements and view settings
- Retrieving curves in the cut plane
- Helper methods and external command mainline
- Sample model and results
- Caveat...

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

-->

### Retrieving Section View Intersection Cut Geometry

I played around creating a
new [SectionCutGeo add-in](https://github.com/jeremytammik/SectionCutGeo) to
retrieve the geometry resulting from cutting a family instance in a section view.

This was prompted
by Martin von Kessel, [@Hacklberg](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/3883640), in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [getting intersection lines from cut in section view](https://forums.autodesk.com/t5/revit-api-forum/how-to-receive-intersection-of-section-and-familyinstance/m-p/8802202):

- [Intersection of section and family instance task](#2) 
- [Solution options, geometry elements and view settings](#3) 
- [Retrieving curves in the cut plane](#4) 
- [Helper methods and external command mainline](#5) 
- [Sample model and results](#6) 
- [Caveat](#7) 


####<a name="2"></a> Intersection of Section and Family Instance Task

**Question:** I am trying to get the Intersection of Section and `FamilyInstance`.

The `GeometryElement` of the family returns all faces, even if they are outside of the section.

Here is what I am after:

<center>
<img src="img/section_cut_geo_query.png" alt="Section cut request" width="549">
</center>

Is there any predefined function to receive the Solid or BoundingBox inside the section? 


####<a name="3"></a> Solution Options, Geometry Elements and View Settings

It is definitely possible, as was explained by Saeed Karshenas in his demonstration
of [visualising section view geometry in OpelGL](https://thebuildingcoder.typepad.com/blog/2011/08/section-view-geometry.html).

To achieve this, you need to specify the section view in the options passed in to get the geometry:

<pre class="code">
  <span style="color:#2b91af;">Options</span>&nbsp;option&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Options</span>();
  option.View&nbsp;=&nbsp;viewSection;
   
  <span style="color:#2b91af;">GeometryElement</span>&nbsp;geometryElement&nbsp;=&nbsp;familyInstance
  &nbsp;&nbsp;.get_Geometry(&nbsp;option&nbsp;);
   
  <span style="color:#2b91af;">GeometryInstance</span>?&nbsp;gInst&nbsp;=&nbsp;geometryElement.First()&nbsp;
  &nbsp;&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">GeometryInstance</span>;
   
  <span style="color:#2b91af;">GeometryElement</span>&nbsp;gSymbol&nbsp;=&nbsp;gInst.GetInstanceGeometry();
</pre>

The crucial point, however, if you want to retrieve the intersection lines of the family instance and the section cut plane, is to set the `VIEWER_BOUND_FAR_CLIPPING` parameter appropriately.

The default setting is `2`, 'Clip without line'.

I changed it to `1`, 'Clip with line', and then the intersection lines are returned:

<pre class="code">
  viewSection.get_Parameter(&nbsp;
  &nbsp;&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.VIEWER_BOUND_FAR_CLIPPING&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;.Set(&nbsp;1&nbsp;);
   
  viewSection.DetailLevel&nbsp;=&nbsp;<span style="color:#2b91af;">ViewDetailLevel</span>.Fine;
</pre>

<center>
<img src="img/section_view_far_clipping.png" alt="Section view far clipping setting" width="279">
</center>


####<a name="4"></a> Retrieving Curves in the Cut Plane

In the SectionCutGeo add-in, I only want to retrieve the intersection curves of the family instance with the cut plane of the section view.

The geometry returned includes the entire truncated solid.

How to determine which of its curves lie in a specific plane?

Initially, I simply called `NewModelCurve`, specifying the sketch plane in that cut plane, for each of the curves I retrieved from the geometry.

It works for the curves that lie in that plane and throws an exception for all the ones that do not:

- An exception of type *Autodesk.Revit.Exceptions.ArgumentException* occurred in RevitAPI.dll;
Additional information: Curve must be in the plane

Throwing all those exceptions is a very horrible thing to do and costs a huge amount of time.

It provides a brain-dead simple way to get rid of the off-plane curves, though.

So far, we simply tried to create a model curve in the given plane. This throws an exception if the curve does not lie in the plane. That is bad and costs a huge amount of performance. Better would be to programmatically check whether the curve lies in the plane beforehand, instead of throwing an exception. How can we determine whether a curve lies in a plane?

I asked the development team whether they can suggest a method to check beforehand whether a curve lies in a given plane or not:

Jeremy: Is there a way to check whether a `Curve` lies in a given `Plane`? I can simply call `NewModelCurve` on it. That throws exception if the curve does not lie in the plane. However, it would be much cleaner and more efficient to check that programmatically instead. Does the Revit API offer anything that I could use for that check, or is this a case of DIY?

Devteam: `Face` has `Intersect(Curve)`.
`Plane` (or other `Surface`) does not.
Could you project the curve end-points (and one intermediate point) with `Surface.Project` and check the distances?  
Alternatively, can’t you build a plane via 3 points on the curve, confirm matching normal vectors and then check on point on the target plane for intersection with the generated plane? Also, check that the initiating curve is planar (no idea how in Revit API but non-planar 3D splines are a thing that ruins all of the above).

Jeremy: Yes, I could project and test the distance. I think I only have arcs and lines, though, so it may be easier to implement my own is-curve-in-plane predicate, much faster, probably... I think the Revit API once boasted a method on the `Curve` class to return the plane it was lying in, but that seems to have vanished...

I added a counter to track all the different kinds of geometry elements I am handling, and it reported:

<pre>
    3 Element
    2 FamilyInstance
    7 GeometryElement
    2 GeometryInstance
  261 Line
    2 null
    9 Solid
    2 Wall
</pre>

Apparently, all the curves I have to handle are `Line` objects, and that is easy using `Plane.Project`:

<pre class="code">
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;Predicate&nbsp;returning&nbsp;true&nbsp;if&nbsp;the&nbsp;given&nbsp;line</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;lies&nbsp;in&nbsp;the&nbsp;given&nbsp;plane</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
  <span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;IsLineInPlane(
  &nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;line,
  &nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;plane&nbsp;)
  {
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p0&nbsp;=&nbsp;line.GetEndPoint(&nbsp;0&nbsp;);
  &nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p1&nbsp;=&nbsp;line.GetEndPoint(&nbsp;1&nbsp;);
  &nbsp;&nbsp;<span style="color:#2b91af;">UV</span>&nbsp;uv0,&nbsp;uv1;
  &nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;d0,&nbsp;d1;
   
  &nbsp;&nbsp;plane.Project(&nbsp;p0,&nbsp;<span style="color:blue;">out</span>&nbsp;uv0,&nbsp;<span style="color:blue;">out</span>&nbsp;d0&nbsp;);
  &nbsp;&nbsp;plane.Project(&nbsp;p1,&nbsp;<span style="color:blue;">out</span>&nbsp;uv1,&nbsp;<span style="color:blue;">out</span>&nbsp;d1&nbsp;);
   
  &nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;_eps&nbsp;&gt;&nbsp;<span style="color:#2b91af;">Math</span>.Abs(&nbsp;d0&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;&amp;&amp;&nbsp;_eps&nbsp;&gt;&nbsp;<span style="color:#2b91af;">Math</span>.Abs(&nbsp;d1&nbsp;);
  }
</pre>

Jeremy: I wonder if `d0` and `d1` are guaranteed to be positive in any case?
Or can they be negative also?
The documentation should tell me, but it does not.

Devteam: The distance output will always be non-negative (up to floating-point rounding error).
You are correct in saying that the API documentation should state that explicitly, though it probably uses "distance" to generally mean unsigned distance.
You're also correct in saying that it would be helpful for Revit's API to provide a function to determine if a curve lies in a plane (or more generally, on any given surface).
There is an internal function that does so, but it's not exposed via the API.


####<a name="5"></a> Helper Methods and External Command Mainline

Using my own curve-in-plane predicate to eliminate all curves not lying in the plane instead of throwing an exception for each one sped up execution tremendously.

It also enabled me to separate the curve collection and the model curve creation from each other.

I implemented a read-only curve collection section followed by a separate, later, transaction for the model curve creation.

The code now looks like this:

<pre class="code">
<span style="color:blue;">namespace</span>&nbsp;SectionCutGeo
{
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;A&nbsp;class&nbsp;to&nbsp;count&nbsp;and&nbsp;report&nbsp;the&nbsp;</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;number&nbsp;of&nbsp;objects&nbsp;encountered.</span>
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">JtObjCounter</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Dictionary</span>&lt;<span style="color:blue;">string</span>,&nbsp;<span style="color:blue;">int</span>&gt;
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Count&nbsp;a&nbsp;new&nbsp;occurence&nbsp;of&nbsp;an&nbsp;object</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Increment(&nbsp;<span style="color:blue;">object</span>&nbsp;obj&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;key&nbsp;=&nbsp;<span style="color:blue;">null</span>&nbsp;==&nbsp;obj
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;?&nbsp;<span style="color:#a31515;">&quot;null&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;:&nbsp;obj.GetType().Name;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;!ContainsKey(&nbsp;key&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Add(&nbsp;key,&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++<span style="color:blue;">this</span>[key];
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Report&nbsp;the&nbsp;number&nbsp;of&nbsp;objects&nbsp;encountered.</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Print()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;keys&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;(&nbsp;Keys&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;keys.Sort();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">string</span>&nbsp;key&nbsp;<span style="color:blue;">in</span>&nbsp;keys&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;<span style="color:#a31515;">&quot;{0,5}&nbsp;{1}&quot;</span>,&nbsp;<span style="color:blue;">this</span>[key],&nbsp;key&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
 
&nbsp;&nbsp;[<span style="color:#2b91af;">Transaction</span>(&nbsp;<span style="color:#2b91af;">TransactionMode</span>.Manual&nbsp;)]
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Command</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IExternalCommand</span>
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Maximum&nbsp;distance&nbsp;for&nbsp;line&nbsp;to&nbsp;be&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;considered&nbsp;to&nbsp;lie&nbsp;in&nbsp;plane</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">const</span>&nbsp;<span style="color:blue;">double</span>&nbsp;_eps&nbsp;=&nbsp;1.0e-6;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;User&nbsp;instructions&nbsp;for&nbsp;running&nbsp;this&nbsp;external&nbsp;command</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">const</span>&nbsp;<span style="color:blue;">string</span>&nbsp;_instructions&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Please&nbsp;launch&nbsp;this&nbsp;&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:#a31515;">&quot;command&nbsp;in&nbsp;a&nbsp;section&nbsp;view&nbsp;with&nbsp;fine&nbsp;level&nbsp;of&nbsp;&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:#a31515;">&quot;detail&nbsp;and&nbsp;far&nbsp;bound&nbsp;clipping&nbsp;set&nbsp;to&nbsp;&#39;Clip&nbsp;with&nbsp;line&#39;&quot;</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Predicate&nbsp;returning&nbsp;true&nbsp;if&nbsp;the&nbsp;given&nbsp;line&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;lies&nbsp;in&nbsp;the&nbsp;given&nbsp;plane</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;IsLineInPlane(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Line</span>&nbsp;line,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;plane&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p0&nbsp;=&nbsp;line.GetEndPoint(&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p1&nbsp;=&nbsp;line.GetEndPoint(&nbsp;1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UV</span>&nbsp;uv0,&nbsp;uv1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;d0,&nbsp;d1;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;plane.Project(&nbsp;p0,&nbsp;<span style="color:blue;">out</span>&nbsp;uv0,&nbsp;<span style="color:blue;">out</span>&nbsp;d0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;plane.Project(&nbsp;p1,&nbsp;<span style="color:blue;">out</span>&nbsp;uv1,&nbsp;<span style="color:blue;">out</span>&nbsp;d1&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;0&nbsp;&lt;=&nbsp;d0,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;expected&nbsp;non-negative&nbsp;distance&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;0&nbsp;&lt;=&nbsp;d1,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;expected&nbsp;non-negative&nbsp;distance&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(&nbsp;_eps&nbsp;&gt;&nbsp;d0&nbsp;)&nbsp;&amp;&amp;&nbsp;(&nbsp;_eps&nbsp;&gt;&nbsp;d1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;Recursively&nbsp;handle&nbsp;geometry&nbsp;element</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;GetCurvesInPlane(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;curves,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">JtObjCounter</span>&nbsp;geoCounter,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;plane,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geo&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter.Increment(&nbsp;geo&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;geo&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">GeometryObject</span>&nbsp;obj&nbsp;<span style="color:blue;">in</span>&nbsp;geo&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter.Increment(&nbsp;obj&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;sol&nbsp;=&nbsp;obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Solid</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;sol&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">EdgeArray</span>&nbsp;edges&nbsp;=&nbsp;sol.Edges;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Edge</span>&nbsp;edge&nbsp;<span style="color:blue;">in</span>&nbsp;edges&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;curve&nbsp;=&nbsp;edge.AsCurve();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;curve&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:#2b91af;">Line</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;we&nbsp;currently&nbsp;only&nbsp;support&nbsp;lines&nbsp;here&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter.Increment(&nbsp;curve&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;IsLineInPlane(&nbsp;curve&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Line</span>,&nbsp;plane&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;curves.Add(&nbsp;curve&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryInstance</span>&nbsp;inst&nbsp;=&nbsp;obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">GeometryInstance</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;inst&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GetCurvesInPlane(&nbsp;curves,&nbsp;geoCounter,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;plane,&nbsp;inst.GetInstanceGeometry()&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;<span style="color:blue;">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;unsupported&nbsp;geometry&nbsp;object&nbsp;&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;obj.GetType().Name&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Result</span>&nbsp;Execute(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">string</span>&nbsp;message,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementSet</span>&nbsp;elements&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UIApplication</span>&nbsp;uiapp&nbsp;=&nbsp;commandData.Application;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UIDocument</span>&nbsp;uidoc&nbsp;=&nbsp;uiapp.ActiveUIDocument;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Application</span>&nbsp;app&nbsp;=&nbsp;uiapp.Application;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;=&nbsp;uidoc.Document;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">View</span>&nbsp;section_view&nbsp;=&nbsp;commandData.View;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Parameter</span>&nbsp;p&nbsp;=&nbsp;section_view.get_Parameter(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.VIEWER_BOUND_FAR_CLIPPING&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:#2b91af;">ViewType</span>.Section&nbsp;!=&nbsp;section_view.ViewType
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||&nbsp;<span style="color:#2b91af;">ViewDetailLevel</span>.Fine&nbsp;!=&nbsp;section_view.DetailLevel
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||&nbsp;1&nbsp;!=&nbsp;p.AsInteger()&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;message&nbsp;=&nbsp;_instructions;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Failed;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;a
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;doc,&nbsp;section_view.Id&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Options</span>&nbsp;opt&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Options</span>()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ComputeReferences&nbsp;=&nbsp;<span style="color:blue;">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IncludeNonVisibleObjects&nbsp;=&nbsp;<span style="color:blue;">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;View&nbsp;=&nbsp;section_view
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SketchPlane</span>&nbsp;plane1&nbsp;=&nbsp;section_view.SketchPlane;&nbsp;<span style="color:green;">//&nbsp;this&nbsp;is&nbsp;null</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Plane</span>&nbsp;plane2&nbsp;=&nbsp;<span style="color:#2b91af;">Plane</span>.CreateByNormalAndOrigin(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;section_view.ViewDirection,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;section_view.Origin&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">JtObjCounter</span>&nbsp;geoCounter&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JtObjCounter</span>();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;&nbsp;curves&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">Curve</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;e&nbsp;<span style="color:blue;">in</span>&nbsp;a&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter.Increment(&nbsp;e&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geo&nbsp;=&nbsp;e.get_Geometry(&nbsp;opt&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GetCurvesInPlane(&nbsp;curves,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter,&nbsp;plane2,&nbsp;geo&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;<span style="color:#a31515;">&quot;Objects&nbsp;analysed:&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;geoCounter.Print();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;{0}&nbsp;cut&nbsp;geometry&nbsp;lines&nbsp;found&nbsp;in&nbsp;section&nbsp;plane.&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;curves.Count&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;tx&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;doc&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.Start(&nbsp;<span style="color:#a31515;">&quot;Create&nbsp;Section&nbsp;Cut&nbsp;Model&nbsp;Curves&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SketchPlane</span>&nbsp;plane3&nbsp;=&nbsp;<span style="color:#2b91af;">SketchPlane</span>.Create(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;doc,&nbsp;plane2&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;c&nbsp;<span style="color:blue;">in</span>&nbsp;curves&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;doc.Create.NewModelCurve(&nbsp;c,&nbsp;plane3&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.Commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}
</pre>

Note that the code listed above is a momentary snapshot at the time of writing.

For the full Visual Studio solution and updates to the code, please refer to
the [SectionCutGeo GitHub repository](https://github.com/jeremytammik/SectionCutGeo).


####<a name="6"></a> Sample Model and Results

Sample model 3D view:

![Sample model 3D view](img/section_cut_geo_3d.png)

Plan view showing section location:

![Plan view showing section location](img/section_cut_geo_plan.png)

Cut geometry in section view:

![Cut geometry in section view](img/section_cut_geo_cut.png)

Model lines representing the cut geometry of the window family instance produced by the add-in in section view:

![Model lines representing cut geometry in section view](img/section_cut_geo_cut_geo_window.png)

Model lines representing the cut geometry of walls, door and window isolated in 3D view:

![Model lines representing cut geometry isolated in 3D view](img/section_cut_geo_cut_geo_3d.png)

Listing the number of processed elements, geometry objects and curves actually lying in the cut plane for this sample:

<pre>
  Objects analysed:
      3 Element
      2 FamilyInstance
      7 GeometryElement
      2 GeometryInstance
    261 Line
      2 null
      9 Solid
      2 Wall

  77 cut geometry lines found in section plane.
</pre>


####<a name="7"></a> Caveat

This sample currently only handles solid and instance geometry objects.

There may well be other object types that need to be handled as well to provide full coverage for all situations.
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

- 13642689 [Mathematical Translations]
  https://forums.autodesk.com/t5/revit-api-forum/mathematical-translations/m-p/7580510
  automatically create walls in cubical family instances

 #RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon http://bit.ly/cloudmodelregen

Alexander Ignatovich shares an exceedingly elegant solution for automatic wall creation, illustrating a number of important concepts and implementing the following functionality very succinctly indeed
&ndash; Retrieve all the <code>cube</code> family instances
&ndash; Retrieve their <code>height</code> parameter value
&ndash; Retrieve their solids making use of the .NET <code>yield</code> operator
&ndash; Extract their horizontal outline contours using an <code>ExtrusionAnalyzer</code>
&ndash; Create walls along each contour curve segment
&ndash; Place a door family instance at the midpoint of each wall...

--->

### Automatic Wall Creation

I highlight yet another thread from 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
on [mathematical translations for automatic wall creation](https://forums.autodesk.com/t5/revit-api-forum/mathematical-translations/m-p/7580510),
with an exceedingly elegant solution
by Alexander [@aignatovich](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/1257478) Ignatovich.

Alexander's macro illustrates a number of important concepts and implements the following functionality very succinctly indeed:

- Retrieve all the `cube` family instances
- Retrieve their `height` parameter value
- Retrieve their solids making use of the .NET `yield` operator
- Extract their horizontal outline contours using an `ExtrusionAnalyzer`
- Create walls along each contour curve segment
- Place a door family instance at the midpoint of each wall

His code is well worth reading in detail!

An absolute must, actually!

**Question:** I am building an add-in that creates walls and doors inside a custom family that acts somewhat as an empty area if you will.

Since I cannot nest walls within the custom family, I have to create them via the API with (x,y,z) coordinates.

When the user places the custom cube (area) family, the walls get built.

The issue is when the user selects a rotation that is not 0 deg, I do not know the translation of coordinates to assign to the wall start and end points.

I have completed all the trig to determine the angles and it will account for any angle the user selects.

How do I do the Mathematical translations to determine a formula which will help me determine the new start and end points of the walls?

The attached document [efortune_trigrevitapi.pdf](zip/autowall_efortune_trigrevitapi.pdf) shows how I determined the angles using the Bounding Box Min and Max along with the insertion point.


**Answer:** Your 5-page PDF listing dozens of formulas may be unnecessary complicated.
 
As far as I can tell from your verbal description, your problem is small and simple.
 
I have a hunch that the solution is smaller and simpler still.
 
I presume that an easy solution can be found that is valid for all quadrants, without distinction.
 
A straight vertical wall depends on only two points, p and q, representing the start and end point of the wall.
 
A cube is inserted into the model, defining a transform T consisting of a rotation by an angle `a` and a translation by a vector `v`.
 
To transform p and q by T, you could first rotate them both around the origin (or whatever axis point you prefer) by `a` and then translate then by `v` to wherever they are supposed to end up.

Alternatively, use the real-world coordinates from the cube vertices as they are.
 
Look at this code and the macro in the attached Revit file [autowalls.rvt](zip/autowall_autowalls.rvt); maybe it will be useful for you:

<pre class="code">
<span style="color:blue;">using</span>&nbsp;System.Collections.Generic;
<span style="color:blue;">using</span>&nbsp;System.Linq;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.Attributes;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.DB;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.DB.Structure;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.Exceptions;
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.UI;
 
<span style="color:blue;">namespace</span>&nbsp;AutoWallsByCubes
{
&nbsp;&nbsp;[<span style="color:#2b91af;">Transaction</span>(&nbsp;<span style="color:#2b91af;">TransactionMode</span>.Manual&nbsp;)]
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">CreateWallsAutomaticallyCommand</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;:&nbsp;<span style="color:#2b91af;">IExternalCommand</span>
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Result</span>&nbsp;Execute(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ExternalCommandData</span>&nbsp;commandData,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">string</span>&nbsp;message,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementSet</span>&nbsp;elements&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;uiapp&nbsp;=&nbsp;commandData.Application;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;uidoc&nbsp;=&nbsp;uiapp.ActiveUIDocument;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;doc&nbsp;=&nbsp;uidoc.Document;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;cubes&nbsp;=&nbsp;FindCubes(&nbsp;doc&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;transaction&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(&nbsp;doc&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;transaction.Start(&nbsp;<span style="color:#a31515;">&quot;create&nbsp;walls&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;cube&nbsp;<span style="color:blue;">in</span>&nbsp;cubes&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;countours&nbsp;=&nbsp;FindCountors(&nbsp;cube&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.SelectMany(&nbsp;x&nbsp;=&gt;&nbsp;x&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;height&nbsp;=&nbsp;cube.LookupParameter(&nbsp;<span style="color:#a31515;">&quot;height&quot;</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AsDouble();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;countour&nbsp;<span style="color:blue;">in</span>&nbsp;countours&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;wall&nbsp;=&nbsp;CreateWall(&nbsp;cube,&nbsp;countour,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;height&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CreateDoor(&nbsp;wall&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;transaction.Commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">Wall</span>&nbsp;CreateWall(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FamilyInstance</span>&nbsp;cube,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Curve</span>&nbsp;curve,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;height&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;doc&nbsp;=&nbsp;cube.Document;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;wallTypeId&nbsp;=&nbsp;doc.GetDefaultElementTypeId(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementTypeGroup</span>.WallType&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Wall</span>.Create(&nbsp;doc,&nbsp;curve.CreateReversed(),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wallTypeId,&nbsp;cube.LevelId,&nbsp;height,&nbsp;0,&nbsp;<span style="color:blue;">false</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">false</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;CreateDoor(&nbsp;<span style="color:#2b91af;">Wall</span>&nbsp;wall&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;locationCurve&nbsp;=&nbsp;(<span style="color:#2b91af;">LocationCurve</span>)&nbsp;wall.Location;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;position&nbsp;=&nbsp;locationCurve.Curve.Evaluate(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0.5,&nbsp;<span style="color:blue;">true</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;document&nbsp;=&nbsp;wall.Document;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;level&nbsp;=&nbsp;(<span style="color:#2b91af;">Level</span>)&nbsp;document.GetElement(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wall.LevelId&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;symbolId&nbsp;=&nbsp;document.GetDefaultFamilyTypeId(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ElementId</span>(&nbsp;<span style="color:#2b91af;">BuiltInCategory</span>.OST_Doors&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;symbol&nbsp;=&nbsp;(<span style="color:#2b91af;">FamilySymbol</span>)&nbsp;document.GetElement(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;symbolId&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;!symbol.IsActive&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;symbol.Activate();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;document.Create.NewFamilyInstance(&nbsp;position,&nbsp;symbol,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wall,&nbsp;level,&nbsp;<span style="color:#2b91af;">StructuralType</span>.NonStructural&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">FamilyInstance</span>&gt;&nbsp;FindCubes(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;collector&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;doc&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;collector
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfCategory(&nbsp;<span style="color:#2b91af;">BuiltInCategory</span>.OST_GenericModel&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">FamilyInstance</span>&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfType&lt;<span style="color:#2b91af;">FamilyInstance</span>&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Where(&nbsp;x&nbsp;=&gt;&nbsp;x.Symbol.FamilyName&nbsp;==&nbsp;<span style="color:#a31515;">&quot;cube&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">CurveLoop</span>&gt;&nbsp;FindCountors(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FamilyInstance</span>&nbsp;familyInstance&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;GetSolids(&nbsp;familyInstance&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.SelectMany(&nbsp;x&nbsp;=&gt;&nbsp;GetCountours(&nbsp;x,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;familyInstance&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">Solid</span>&gt;&nbsp;GetSolids(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;element&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;geometry&nbsp;=&nbsp;element
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.get_Geometry(&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Options</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ComputeReferences&nbsp;=&nbsp;<span style="color:blue;">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IncludeNonVisibleObjects&nbsp;=&nbsp;<span style="color:blue;">true</span>&nbsp;}&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;geometry&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Enumerable</span>.Empty&lt;<span style="color:#2b91af;">Solid</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;GetSolids(&nbsp;geometry&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Where(&nbsp;x&nbsp;=&gt;&nbsp;x.Volume&nbsp;&gt;&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">Solid</span>&gt;&nbsp;GetSolids(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">GeometryObject</span>&gt;&nbsp;geometryElement&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;geometry&nbsp;<span style="color:blue;">in</span>&nbsp;geometryElement&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;solid&nbsp;=&nbsp;geometry&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Solid</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;solid&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">yield</span>&nbsp;<span style="color:blue;">return</span>&nbsp;solid;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;instance&nbsp;=&nbsp;geometry&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">GeometryInstance</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;instance&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;instanceSolid&nbsp;<span style="color:blue;">in</span>&nbsp;GetSolids(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;instance.GetInstanceGeometry()&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">yield</span>&nbsp;<span style="color:blue;">return</span>&nbsp;instanceSolid;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;element&nbsp;=&nbsp;geometry&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">GeometryElement</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;element&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">var</span>&nbsp;elementSolid&nbsp;<span style="color:blue;">in</span>&nbsp;GetSolids(&nbsp;element&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">yield</span>&nbsp;<span style="color:blue;">return</span>&nbsp;elementSolid;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">CurveLoop</span>&gt;&nbsp;GetCountours(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solid,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;element&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">try</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;plane&nbsp;=&nbsp;<span style="color:#2b91af;">Plane</span>.CreateByNormalAndOrigin(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>.BasisZ,&nbsp;element.get_BoundingBox(&nbsp;<span style="color:blue;">null</span>&nbsp;).Min&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;analyzer&nbsp;=&nbsp;<span style="color:#2b91af;">ExtrusionAnalyzer</span>.Create(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;solid,&nbsp;plane,&nbsp;<span style="color:#2b91af;">XYZ</span>.BasisZ&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;face&nbsp;=&nbsp;analyzer.GetExtrusionBase();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;face.GetEdgesAsCurveLoops();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">catch</span>(&nbsp;Autodesk.Revit.Exceptions
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.<span style="color:#2b91af;">InvalidOperationException</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Enumerable</span>.Empty&lt;<span style="color:#2b91af;">CurveLoop</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}
</pre>

Cube family instances before running command:

<center>
<img src="img/autowall_before.png" alt="Cubes" width="400"/>
</center>

Walls and doors added along cube horizontal contours after running command:

<center>
<img src="img/autowall_after.png" alt="Walls and doors added along cube faces" width="400"/>
</center>

If you prefer the other approach, explore
the [Transform](http://thebuildingcoder.typepad.com/blog/2009/03/transform.html) class
and related concepts.

You can get the transform of your family instance using the `familyInstance.GetTotalTransform` method.

Many thanks to Alexander for this beautiful and efficient sample code!

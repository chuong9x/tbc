<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- img/adam_jeremy_luis_alvaro_petr_manrique_aecom_1200x733.jpg
  Among the participant are Álvaro Pérez, Manrique Gómez and Luis López of the [AECOM](https://www.aecom.com) BIM team in Madrid, who say:
  > We have been developing desktop solutions for Revit for years and are currently moving to the cloud and the Forge platform.
  > The experiences at this Forge accelerator in Barcelona are invaluable for us thanks to all the support from the Autodesk guys.
  Posing for us in the background, you can admire the Autodesk Forge table-tennis experts Adam and Petr.

- IFC exporter utilities enable you to add your own IFC related built-in parameters to elements
  how to [add an IFC_GUID BuiltInParameter to a beam that doesn't contain such parameter](https://forums.autodesk.com/t5/revit-api-forum/add-a-ifc-guid-builtinparameter-to-a-beam-that-doesn-t-contain/m-p/8870228)

twitter:

&ndash;
...

linkedin:

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

Before diving deeper into the Forge acceleratr topics

-->

### Room Closed Shell DirectShape for Forge Viewer

I explored three main topics here at the Forge accelerator:

- Room closed shell solid visibility in the Forge viewer
- Rebar simplification: replace rebar elements with simplified solids or model curves
- [glTF](https://en.wikipedia.org/wiki/GlTF) export

Today, I'll dive deeper into the first:

- [IFC exporter utility adds new built-in parameter](#2)
- [Barcelona Forge accelerator](#3)
- [Room closed shell in the Forge viewer](#4)
- [Triangulate the solid face by face](#5)
- [Triangulate entire solid](#6)
- [Tessellation accuracy control documentation error](#7)


####<a name="2"></a> IFC Exporter Utility Adds New Built-In Parameter

Before that, however, here is an interesting topic from
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160).

Quite surprisingly, for me, the IFC exporter utilities enable you to add your own new IFC related built-in parameter to an element.

I was previously not aware of any way to add new built-in parameters to existing elements lacking them.

This solution was discovered and shared by Luis González Torquemada in his thread
on [adding an IFC_GUID BuiltInParameter to a beam that doesn't contain such a parameter](https://forums.autodesk.com/t5/revit-api-forum/add-a-ifc-guid-builtinparameter-to-a-beam-that-doesn-t-contain/m-p/8870228):

**Question:** I've created a beam by

<pre class="code">
  <span style="color:#2b91af;">FamilyInstance</span>&nbsp;instance&nbsp;
  &nbsp;&nbsp;=&nbsp;document.Create.NewFamilyInstance(&nbsp;beamLine,&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;famSymbol,&nbsp;level,&nbsp;<span style="color:#2b91af;">StructuralType</span>.Beam&nbsp;);
</pre>

Now I need to set a particular `IFC_GUID` value to this element (it comes from another application), but

<pre class="code">
&nbsp;&nbsp;<span style="color:#2b91af;">Parameter</span>&nbsp;ifcParam&nbsp;=&nbsp;instance.get_Parameter(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.IFC_GUID&nbsp;);
</pre>

returns null.

How can I add this parameter (some other elements in the same document have this parameter)?

The issue I want to solve with this question is:

When Revit imports a beam from an IFC file, the `StructuralType` of the `Element` (or `FamilyInstance`) is `StructuralType.NonStructural` instead of `StructuralType.Beam`.
So, you can’t assign it any family symbol of `BuiltInCategory.OST_StructuralFraming` (not via API, nor via the Revit menu).
So, I must delete the element and create a new one with the correct `StructuralType.Beam` and the same `IFC_GUID`.

Note that this issue doesn’t arise with structural columns.

**Answer:** I do not believe you can add a built-in parameter to an element that does not have it already.

**Response:** I’ve found it! The solution is:

<pre class="code">
&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;eid&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ElementId</span>(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.IFC_GUID&nbsp;);

&nbsp;&nbsp;Autodesk.Revit.DB.IFC.ExporterIFCUtils.AddValueString(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;instance,&nbsp;eid,&nbsp;ifcguid&nbsp;);
</pre>

If you create a new beam with the API, the `Parameters` property of the beam doesn't contain a IFC_GUID member, and get_Parameter(BuiltInParameter.IFC_GUID) returns null.

After using the code above, get_Parameter(BuiltInParameter.IFC_GUID) returns correctly and Revit shows it in the properties pane.

By the way, can you tell me how can I change the StructuralType property of an Element from NonStructural to Beam?

**Answer:** I did not know (or believe!) that you can add a new built-in parameter to an element.

So that functionality is rather special.

Unfortunately,
the [StructuralType property is read-only](https://www.revitapidocs.com/2020/6b76b6e9-b334-bae7-bd74-02273f6db108.htm),
so you cannot change it:

I guess the only way to define it is during creation.

So, you are doing the right thing creating new beams.


####<a name="3"></a> Barcelona Forge Accelerator

Traditionally, Thursday evening is the time for the Forge accelerator celebratory dinner:

<center>
<img src="img/group_photo_1_1050x525.png" alt="Forge accelerator celebratory dinner in Marina Bay" width="525">
</center>

Among our participants we welcome Álvaro Pérez, Manrique Gómez and Luis López of
the [AECOM](https://www.aecom.com) BIM team in Madrid, who say:

> We have been developing desktop solutions for Revit for years and are currently moving to the cloud and the Forge platform.

> The experiences at this Forge accelerator in Barcelona are invaluable for us thanks to all the support from the Autodesk guys.

<center>
<img src="img/adam_jeremy_luis_alvaro_petr_manrique_aecom_1200x733.jpg" alt="AECOM at the Forge accelerator" width="600">
</center>

Posing for us in the background, you can also admire the table-tennis skills of Forge experts Adam and Petr:


####<a name="4"></a> Room Closed Shell in the Forge Viewer

I recently implemented an external command
that [creates `DirectShape` elements to represent room volumes](https://thebuildingcoder.typepad.com/blog/2019/05/generate-directshape-element-to-represent-room-volume.html).
The code is available in
the [RoomVolumeDirectShape GitHub repository](https://github.com/jeremytammik/RoomVolumeDirectShape).

The initial implementation was very simple, since the `Room.GetClosedShell` method returns a `GeometryElement` that can be passed straight into the direct shape `SetShape` method with no further ado.

Unfortunately, on further testing, we discovered that the resulting generic model direct shape elements do not show up as expected in the viewer.

No geometry is displayed for it, and it is not even listed in the model browser.

Here is a Forge view of a room. It contains a direct shape representing the room volume. It appears perfectly fine in Revit. However, in the Forge viewer, no trace of it is visible:

<center>
<img src="img/room_closed_shell_forge.png" alt="Direct shape missing in Forge" width="450">
</center>

This led to a lot of head scratching and further research.

Apparently, the solid returned by the Revit API for the room closed shell is flawed in some way.

I explored a number of approaches to fix it, then took recourse to recreating the entire solid from scratch in various ways, two of which turned out to produce reliable results so far:

- Fix it somehow
- Triangulate face by face
- Recreate solid using `EdgeLoops` &ndash; orientation needs to be fixed
- Recreate solid using `GetEdgesAsCurveLoops` &ndash; orientation needs to be fixed
- Triangulate entire solid using the `SolidUtils` `TessellateSolidOrShell` method

Most of my efforts are preserved in the source code, enclosed in `#if` pragmas.


####<a name="5"></a> Triangulate the Solid Face by Face

One reliable way to generate a valid solid to replace the flawed one produced by `GetClosedShell` is to query each face of the solid for its triangulation and use the triangular facets to define the direct shape instead, as described in the discussion
on [creating a `DirectShape` element from a face mesh](https://thebuildingcoder.typepad.com/blog/2015/09/directshape-from-face-and-sketch-plane-reuse.html)

The following code achieves this:

<pre class="code">
<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;Create&nbsp;a&nbsp;new&nbsp;list&nbsp;of&nbsp;geometry&nbsp;objects&nbsp;from&nbsp;the&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;given&nbsp;input.&nbsp;As&nbsp;input,&nbsp;we&nbsp;supply&nbsp;the&nbsp;result&nbsp;of&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;Room.GetClosedShell.&nbsp;The&nbsp;output&nbsp;is&nbsp;the&nbsp;exact&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;same&nbsp;solid&nbsp;lacking&nbsp;whatever&nbsp;flaws&nbsp;are&nbsp;present&nbsp;</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;in&nbsp;the&nbsp;input&nbsp;solid.</span>
<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IList</span>&lt;<span style="color:#2b91af;">GeometryObject</span>&gt;&nbsp;CopyGeometry(
&nbsp;&nbsp;<span style="color:#2b91af;">GeometryElement</span>&nbsp;geo,
&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;materialId&nbsp;)
{
&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderResult</span>&nbsp;result&nbsp;=&nbsp;<span style="color:blue;">null</span>;

&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilder</span>&nbsp;builder
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilder</span>();

&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;nFaces&nbsp;=&nbsp;0;
&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;nTriangles&nbsp;=&nbsp;0;
&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;&nbsp;vertices&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;(&nbsp;3&nbsp;);

&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">GeometryObject</span>&nbsp;obj&nbsp;<span style="color:blue;">in</span>&nbsp;geo&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solid&nbsp;=&nbsp;obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Solid</span>;

&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;solid&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;0&nbsp;&lt;&nbsp;solid.Volume&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.OpenConnectedFaceSet(&nbsp;<span style="color:blue;">false</span>&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Iterate&nbsp;over&nbsp;the&nbsp;individual&nbsp;solid&nbsp;faces</span>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Face</span>&nbsp;f&nbsp;<span style="color:blue;">in</span>&nbsp;solid.Faces&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Clear();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Mesh</span>&nbsp;mesh&nbsp;=&nbsp;f.Triangulate();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;n&nbsp;=&nbsp;mesh.NumTriangles;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;n;&nbsp;++i&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">MeshTriangle</span>&nbsp;triangle&nbsp;=&nbsp;mesh.get_Triangle(&nbsp;i&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p1&nbsp;=&nbsp;triangle.get_Vertex(&nbsp;0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p2&nbsp;=&nbsp;triangle.get_Vertex(&nbsp;1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;p3&nbsp;=&nbsp;triangle.get_Vertex(&nbsp;2&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;p1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;p2&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;p3&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedFace</span>&nbsp;tf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">TessellatedFace</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices,&nbsp;materialId&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;builder.DoesFaceHaveEnoughLoopsAndVertices(&nbsp;tf&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.AddFace(&nbsp;tf&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++nTriangles;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.AddFace(&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">TessellatedFace</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices,&nbsp;materialId&nbsp;)&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++nFaces;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.CloseConnectedFaceSet();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Target&nbsp;=&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderTarget</span>.AnyGeometry;&nbsp;<span style="color:green;">//&nbsp;Solid&nbsp;failed</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Fallback&nbsp;=&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderFallback</span>.Mesh;&nbsp;<span style="color:green;">//&nbsp;use&nbsp;Abort&nbsp;if&nbsp;target&nbsp;is&nbsp;Solid</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;builder.GetBuildResult();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;result.GetGeometricalObjects();
}
</pre>

Unfortunately, all the triangle edges remain visible in the Revit model, even interior edges lying inside a planar face:

<center>
<img src="img/room_closed_shell_rvt.png" alt="Direct shape defined using triangles" width="10">
</center>

####<a name="6"></a> Triangulate Entire Solid

One serious problem may arise triangulating each face separately as shown above:

If two face meeting in a curved edge are independently triangulated, their respective tessellation of the shared edge may differ, resulting in triangle vertices on one face not matching the ones on the other, causing gaps in the resulting solid.

That can be solved only by triangulating the entire solid in one fell swoop, which can be easily achieved using the `SolidUtils` `TessellateSolidOrShell` method as follows.

This code also includes some traces of initial experiments exporting `glTF` facet data:

<pre class="code">
&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderResult</span>&nbsp;result&nbsp;=&nbsp;<span style="color:blue;">null</span>;

&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilder</span>&nbsp;builder
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilder</span>();

&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;nTriangles&nbsp;=&nbsp;0;
&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;&nbsp;vertices&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">XYZ</span>&gt;(&nbsp;3&nbsp;);

&nbsp;&nbsp;<span style="color:green;">//&nbsp;Collect&nbsp;data&nbsp;for&nbsp;glTF:&nbsp;a&nbsp;list&nbsp;of&nbsp;vertex</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;coordinates&nbsp;in&nbsp;millimetres,&nbsp;and&nbsp;a&nbsp;list&nbsp;of</span>
&nbsp;&nbsp;<span style="color:green;">//&nbsp;triangle&nbsp;vertex&nbsp;indices&nbsp;into&nbsp;the&nbsp;list.</span>

&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;gltfVertexCoordinates&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">int</span>&gt;();
&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;gltfVertexIndices&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">int</span>&gt;();
&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;gltfVertexIndexBase&nbsp;=&nbsp;0;

&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">GeometryObject</span>&nbsp;obj&nbsp;<span style="color:blue;">in</span>&nbsp;geo&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;solid&nbsp;=&nbsp;obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Solid</span>;

&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;<span style="color:blue;">null</span>&nbsp;!=&nbsp;solid&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;0&nbsp;&lt;&nbsp;solid.Volume&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.OpenConnectedFaceSet(&nbsp;<span style="color:blue;">false</span>&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SolidUtils</span>.IsValidForTessellation(&nbsp;solid&nbsp;),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;expected&nbsp;a&nbsp;valid&nbsp;solid&nbsp;for&nbsp;room&nbsp;closed&nbsp;shell&quot;</span>&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SolidOrShellTessellationControls</span>&nbsp;controls
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">SolidOrShellTessellationControls</span>()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Summary:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A&nbsp;positive&nbsp;real&nbsp;number&nbsp;specifying&nbsp;how&nbsp;accurately&nbsp;a&nbsp;triangulation&nbsp;should&nbsp;approximate</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a&nbsp;solid&nbsp;or&nbsp;shell.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Exceptions:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;T:Autodesk.Revit.Exceptions.ArgumentOutOfRangeException:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When&nbsp;setting&nbsp;this&nbsp;property:&nbsp;The&nbsp;given&nbsp;value&nbsp;for&nbsp;accuracy&nbsp;must&nbsp;be&nbsp;greater&nbsp;than</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0&nbsp;and&nbsp;no&nbsp;more&nbsp;than&nbsp;30000&nbsp;feet.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;This&nbsp;statement&nbsp;is&nbsp;not&nbsp;true.&nbsp;I&nbsp;set&nbsp;Accuracy&nbsp;=&nbsp;0.003&nbsp;and&nbsp;an&nbsp;exception&nbsp;was&nbsp;thrown.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Setting&nbsp;it&nbsp;to&nbsp;0.006&nbsp;was&nbsp;acceptable.&nbsp;0.03&nbsp;is&nbsp;a&nbsp;bit&nbsp;over&nbsp;9&nbsp;mm.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Remarks:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The&nbsp;maximum&nbsp;distance&nbsp;from&nbsp;a&nbsp;point&nbsp;on&nbsp;the&nbsp;triangulation&nbsp;to&nbsp;the&nbsp;nearest&nbsp;point&nbsp;on</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;the&nbsp;solid&nbsp;or&nbsp;shell&nbsp;should&nbsp;be&nbsp;no&nbsp;greater&nbsp;than&nbsp;the&nbsp;specified&nbsp;accuracy.&nbsp;This&nbsp;constraint</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;may&nbsp;be&nbsp;approximately&nbsp;enforced.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Accuracy&nbsp;=&nbsp;0.03,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Summary:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;An&nbsp;number&nbsp;between&nbsp;0&nbsp;and&nbsp;1&nbsp;(inclusive)&nbsp;specifying&nbsp;the&nbsp;level&nbsp;of&nbsp;detail&nbsp;for&nbsp;the</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;triangulation&nbsp;of&nbsp;a&nbsp;solid&nbsp;or&nbsp;shell.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Exceptions:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;T:Autodesk.Revit.Exceptions.ArgumentOutOfRangeException:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When&nbsp;setting&nbsp;this&nbsp;property:&nbsp;The&nbsp;given&nbsp;value&nbsp;for&nbsp;levelOfDetail&nbsp;must&nbsp;lie&nbsp;between</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0&nbsp;and&nbsp;1&nbsp;(inclusive).</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Remarks:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Smaller&nbsp;values&nbsp;yield&nbsp;coarser&nbsp;triangulations&nbsp;(fewer&nbsp;triangles),&nbsp;while&nbsp;larger&nbsp;values</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yield&nbsp;finer&nbsp;triangulations&nbsp;(more&nbsp;triangles).</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LevelOfDetail&nbsp;=&nbsp;0.1,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Summary:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A&nbsp;non-negative&nbsp;real&nbsp;number&nbsp;specifying&nbsp;the&nbsp;minimum&nbsp;allowed&nbsp;angle&nbsp;for&nbsp;any&nbsp;triangle</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;in&nbsp;the&nbsp;triangulation,&nbsp;in&nbsp;radians.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Exceptions:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;T:Autodesk.Revit.Exceptions.ArgumentOutOfRangeException:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When&nbsp;setting&nbsp;this&nbsp;property:&nbsp;The&nbsp;given&nbsp;value&nbsp;for&nbsp;minAngleInTriangle&nbsp;must&nbsp;be&nbsp;at</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;least&nbsp;0&nbsp;and&nbsp;less&nbsp;than&nbsp;60&nbsp;degrees,&nbsp;expressed&nbsp;in&nbsp;radians.&nbsp;The&nbsp;value&nbsp;0&nbsp;means&nbsp;to</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ignore&nbsp;the&nbsp;minimum&nbsp;angle&nbsp;constraint.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Remarks:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A&nbsp;small&nbsp;value&nbsp;can&nbsp;be&nbsp;useful&nbsp;when&nbsp;triangulating&nbsp;long,&nbsp;thin&nbsp;objects,&nbsp;in&nbsp;order&nbsp;to</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;keep&nbsp;the&nbsp;number&nbsp;of&nbsp;triangles&nbsp;small,&nbsp;but&nbsp;it&nbsp;can&nbsp;result&nbsp;in&nbsp;long,&nbsp;thin&nbsp;triangles,</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;which&nbsp;are&nbsp;not&nbsp;acceptable&nbsp;for&nbsp;all&nbsp;applications.&nbsp;If&nbsp;the&nbsp;value&nbsp;is&nbsp;too&nbsp;large,&nbsp;this</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;constraint&nbsp;may&nbsp;not&nbsp;be&nbsp;satisfiable,&nbsp;causing&nbsp;the&nbsp;triangulation&nbsp;to&nbsp;fail.&nbsp;This&nbsp;constraint</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;may&nbsp;be&nbsp;approximately&nbsp;enforced.&nbsp;A&nbsp;value&nbsp;of&nbsp;0&nbsp;means&nbsp;to&nbsp;ignore&nbsp;the&nbsp;minimum&nbsp;angle</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;constraint.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinAngleInTriangle&nbsp;=&nbsp;3&nbsp;*&nbsp;<span style="color:#2b91af;">Math</span>.PI&nbsp;/&nbsp;180.0,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Summary:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A&nbsp;positive&nbsp;real&nbsp;number&nbsp;specifying&nbsp;the&nbsp;minimum&nbsp;allowed&nbsp;value&nbsp;for&nbsp;the&nbsp;external</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;angle&nbsp;between&nbsp;two&nbsp;adjacent&nbsp;triangles,&nbsp;in&nbsp;radians.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Exceptions:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;T:Autodesk.Revit.Exceptions.ArgumentOutOfRangeException:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When&nbsp;setting&nbsp;this&nbsp;property:&nbsp;The&nbsp;given&nbsp;value&nbsp;for&nbsp;minExternalAngleBetweenTriangles</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;must&nbsp;be&nbsp;greater&nbsp;than&nbsp;0&nbsp;and&nbsp;no&nbsp;more&nbsp;than&nbsp;30000&nbsp;feet.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Remarks:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A&nbsp;small&nbsp;value&nbsp;yields&nbsp;more&nbsp;smoothly&nbsp;curved&nbsp;triangulated&nbsp;surfaces,&nbsp;usually&nbsp;at&nbsp;the</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;expense&nbsp;of&nbsp;an&nbsp;increase&nbsp;in&nbsp;the&nbsp;number&nbsp;of&nbsp;triangles.&nbsp;Note&nbsp;that&nbsp;this&nbsp;setting&nbsp;has</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;no&nbsp;effect&nbsp;for&nbsp;planar&nbsp;surfaces.&nbsp;This&nbsp;constraint&nbsp;may&nbsp;be&nbsp;approximately&nbsp;enforced.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinExternalAngleBetweenTriangles&nbsp;=&nbsp;0.2&nbsp;*&nbsp;<span style="color:#2b91af;">Math</span>.PI
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TriangulatedSolidOrShell</span>&nbsp;shell
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:#2b91af;">SolidUtils</span>.TessellateSolidOrShell(&nbsp;solid,&nbsp;controls&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;n&nbsp;=&nbsp;shell.ShellComponentCount;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;1&nbsp;==&nbsp;n,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;expected&nbsp;just&nbsp;one&nbsp;shell&nbsp;component&nbsp;in&nbsp;room&nbsp;closed&nbsp;shell&quot;</span>&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TriangulatedShellComponent</span>&nbsp;component
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;shell.GetShellComponent(&nbsp;0&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndexBase&nbsp;=&nbsp;gltfVertexCoordinates.Count;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;=&nbsp;component.VertexCount;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;n;&nbsp;&nbsp;++i&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">XYZ</span>&nbsp;v&nbsp;=&nbsp;component.GetVertex(&nbsp;i&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexCoordinates.Add(&nbsp;FootToMm(&nbsp;v.X&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexCoordinates.Add(&nbsp;FootToMm(&nbsp;v.Y&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexCoordinates.Add(&nbsp;FootToMm(&nbsp;v.Z&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;=&nbsp;component.TriangleCount;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;n;&nbsp;&nbsp;++i&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TriangleInShellComponent</span>&nbsp;t&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;component.GetTriangle(&nbsp;i&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Clear();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;component.GetVertex(&nbsp;t.VertexIndex0&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;component.GetVertex(&nbsp;t.VertexIndex1&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices.Add(&nbsp;component.GetVertex(&nbsp;t.VertexIndex2&nbsp;)&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndices.Add(&nbsp;gltfVertexIndexBase&nbsp;+&nbsp;t.VertexIndex0&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndices.Add(&nbsp;gltfVertexIndexBase&nbsp;+&nbsp;t.VertexIndex1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndices.Add(&nbsp;gltfVertexIndexBase&nbsp;+&nbsp;t.VertexIndex2&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TessellatedFace</span>&nbsp;tf&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">TessellatedFace</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vertices,&nbsp;materialId&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;builder.DoesFaceHaveEnoughLoopsAndVertices(&nbsp;tf&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.AddFace(&nbsp;tf&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++nTriangles;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.CloseConnectedFaceSet();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Target&nbsp;=&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderTarget</span>.AnyGeometry;&nbsp;<span style="color:green;">//&nbsp;Solid&nbsp;failed</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Fallback&nbsp;=&nbsp;<span style="color:#2b91af;">TessellatedShapeBuilderFallback</span>.Mesh;&nbsp;<span style="color:green;">//&nbsp;use&nbsp;Abort&nbsp;if&nbsp;target&nbsp;is&nbsp;Solid</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;builder.GetBuildResult();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Log&nbsp;glTF&nbsp;data</span>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;{0}&nbsp;glTF&nbsp;vertex&nbsp;coordinates&nbsp;in&nbsp;millimetres:&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexCoordinates.Count&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;<span style="color:blue;">string</span>.Join(&nbsp;<span style="color:#a31515;">&quot;&nbsp;&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexCoordinates.Select&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&gt;&nbsp;i.ToString()&nbsp;)&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;{0}&nbsp;glTF&nbsp;triangle&nbsp;vertex&nbsp;indices:&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndices.Count&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Print(&nbsp;<span style="color:blue;">string</span>.Join(&nbsp;<span style="color:#a31515;">&quot;&nbsp;&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gltfVertexIndices.Select&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&gt;&nbsp;i.ToString()&nbsp;)&nbsp;)&nbsp;);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Save&nbsp;glTF&nbsp;data&nbsp;to&nbsp;binary&nbsp;file</span>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">FileStream</span>&nbsp;file&nbsp;=&nbsp;<span style="color:#2b91af;">File</span>.Create(&nbsp;<span style="color:#a31515;">&quot;C:/tmp/&quot;</span>&nbsp;+&nbsp;_gltf_path&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>(&nbsp;<span style="color:#2b91af;">BinaryWriter</span>&nbsp;writer&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">BinaryWriter</span>(&nbsp;file&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;i&nbsp;<span style="color:blue;">in</span>&nbsp;gltfVertexCoordinates&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;writer.Write(&nbsp;(<span style="color:blue;">float</span>)&nbsp;i&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;i&nbsp;<span style="color:blue;">in</span>&nbsp;gltfVertexIndices&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.Assert(&nbsp;<span style="color:blue;">ushort</span>.MaxValue&nbsp;&gt;&nbsp;i,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;expected&nbsp;triangle&nbsp;vertex&nbsp;indices&nbsp;to&nbsp;fit&nbsp;into&nbsp;unsigned&nbsp;short&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;writer.Write(&nbsp;(<span style="color:blue;">ushort</span>)&nbsp;i&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;result.GetGeometricalObjects();
</pre>

Note that this approach enables the use of the `SolidOrShellTessellationControls`, which provide a great degree of control over the tessellation accuracy, something that developers have long been clamouring for.

And, also importantly, as noted above, the result of this call is guaranteed to be a valid watertight closed shell, unlike most other approaches provided by the Revit API.


####<a name="7"></a> Solid or Shell Tessellation Accuracy Control Documentation Error

Among the many issues I faced, this was one of the most trivial ones:

The Revit API documentation of
the [SolidOrShellTessellationControls.Accuracy property](https://www.revitapidocs.com/2020/bf865045-141f-8ef2-0d31-a26f488cad1e.htm) states:

- *The given value for accuracy must be greater than 0 and no more than 30000 feet*.

In my initial test, I specified an accuracy of 0.003, corresponding to ca. 0.9 mm, which clearly fulfils that requirement.

An exception was thrown.

I raised it to 0.006, a bit over 1.8 mm, and it passed.

In the end, as you can see above, I raised it further, to 0.03, a bit over 9 mm.

I assume the true minimum limitation correlates with the Revit minimum model line length limit, which is around 1/16th of an inch.

<pre>
  jc&gt; inch = 25.4

  jc&gt; foot = 12 * inch = 304.8

  jc&gt; minlen = inch / 16 = 1.5875

  jc&gt; accuracy = 0.003

  jc&gt; accuracy * foot = 0.9144

  jc&gt; 0.006 * foot = 1.8288
</pre>

Coming up:

- Rebar simplification: replace rebar elements with simplified solids or model curves
- [glTF](https://en.wikipedia.org/wiki/GlTF) export of room volumes


<center>
<img src="img/group_photo_2_1100x800.png" alt="Forge accelerator celebratory dinner in Marina Bay" width="550">
</center>
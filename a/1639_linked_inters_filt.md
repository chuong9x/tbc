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

- 14040989 [Revit API: intersecting model elements]
  using filtered element intersection in a linked file

- https://forums.autodesk.com/t5/revit-api-forum/linked-file-element-intersection-solid-geometry/m-p/7900208

 #RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon 

We recently discussed filtering for intersecting elements.
Here is a closely related issue with an additional twist
&ndash; Determining elements intersecting mass in a linked file
&ndash; Coding suggestions and transformations
&ndash; Solution by applying transformations...

--->

### Using Intersection Filter with Linked File

We recently 
discussed [filtering for intersecting elements](http://thebuildingcoder.typepad.com/blog/2018/03/create-2d-arc-and-filter-for-intersecting-elements.html).

Here is a closely related issue with an additional twist that I discussed
with Gustav Blom, structural engineer at [Rambøll](http://www.ramboll.no) in Norway:

- [Determining elements intersecting mass in a linked file](#2) 
- [Coding suggestions and transformations](#3) 
- [Solution by applying transformations](#4) 

<center>
<img src="img/ramboell1.png" alt="Rambøll" width="238"/>
</center>


####<a name="2"></a>Determining Elements Intersecting Mass in a Linked File

**Question:** I am implementing an add-in that determines all model elements inside a mass in a linked file *MassModel.rvt*.

I have used a filtered element collector with an `ElementIntersectsSolidFilter` using the solid representation of the mass as argument.

Here is my main filtering code:

<pre class="code">
<span style="color:blue;">Public</span>&nbsp;<span style="color:blue;">Sub</span>&nbsp;Run()
 
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;oWrite1&nbsp;<span style="color:blue;">As</span>&nbsp;IO.<span style="color:#2b91af;">StreamWriter</span>
 
&nbsp;&nbsp;oWrite1&nbsp;=&nbsp;IO.<span style="color:#2b91af;">File</span>.CreateText(<span style="color:#a31515;">&quot;C:/&nbsp;.&nbsp;.&nbsp;.&nbsp;/ObjectLocation.txt&quot;</span>)
 
&nbsp;&nbsp;<span style="color:green;">&#39;&nbsp;Collect&nbsp;linked&nbsp;files</span>
 
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;linkedfilefilter&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementCategoryFilter</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementCategoryFilter</span>(<span style="color:#2b91af;">BuiltInCategory</span>.OST_RvtLinks)
 
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;linkedfilecollector&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(m_doc)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(linkedfilefilter)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType
 
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;options&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Options</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">Options</span>()
 
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;tx&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Transaction</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">Transaction</span>(m_doc)
&nbsp;&nbsp;tx.Start(<span style="color:#a31515;">&quot;Object&nbsp;Locate&quot;</span>)
 
&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;linkedfilecollector.Count&nbsp;&gt;&nbsp;0&nbsp;<span style="color:blue;">Then</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;linkedfile&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">RevitLinkInstance</span>&nbsp;<span style="color:blue;">In</span>&nbsp;linkedfilecollector
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;linkedfile.GetLinkDocument.Title.Equals(<span style="color:#a31515;">&quot;MassModel.rvt&quot;</span>)&nbsp;<span style="color:blue;">Then</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Try</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">&#39;&nbsp;Find&nbsp;all&nbsp;elements&nbsp;with&nbsp;OBJ-LOCATION-RDK&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">&#39;&nbsp;Parameter&nbsp;And&nbsp;erase&nbsp;previous&nbsp;values</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;collector&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(m_doc)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsViewIndependent
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;el&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;<span style="color:blue;">In</span>&nbsp;collector
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;el.Category&nbsp;<span style="color:blue;">IsNot</span>&nbsp;<span style="color:blue;">Nothing</span>&nbsp;<span style="color:blue;">And</span>&nbsp;el.Parameters.Size&nbsp;&gt;&nbsp;0&nbsp;<span style="color:blue;">Then</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;ElementParameters&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ParameterSet</span>&nbsp;=&nbsp;el.Parameters
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;elparam&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Parameter</span>&nbsp;<span style="color:blue;">In</span>&nbsp;ElementParameters
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;elparam.Definition.Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;OBJ-LOCATION-RDK&quot;</span>&nbsp;<span style="color:blue;">Then</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;elparam.Set(<span style="color:#a31515;">&quot;&quot;</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;oWrite1.WriteLine(el.Category.Name)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;linkedmassCatFilter&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementCategoryFilter</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementCategoryFilter</span>(<span style="color:#2b91af;">BuiltInCategory</span>.OST_Mass)
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;massCollector&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(linkedfile.GetLinkDocument)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType()&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(linkedmassCatFilter)
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;populatedcounter&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:blue;">Integer</span>&nbsp;=&nbsp;0
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;mass&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;<span style="color:blue;">In</span>&nbsp;massCollector
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;solSet&nbsp;<span style="color:blue;">As</span>&nbsp;Generic.<span style="color:#2b91af;">IEnumerable</span>(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">Solid</span>)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;mass.Geometry(options).OfType(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">Solid</span>)()
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;potSol&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;<span style="color:blue;">In</span>&nbsp;solSet
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;(potSol&nbsp;<span style="color:blue;">IsNot</span>&nbsp;<span style="color:blue;">Nothing</span>&nbsp;<span style="color:blue;">And</span>&nbsp;<span style="color:blue;">Not</span>&nbsp;potSol.Edges.IsEmpty)&nbsp;<span style="color:blue;">Then</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;elementintersectsolidfilter&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementIntersectsSolidFilter</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementIntersectsSolidFilter</span>(potSol)
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;intersectingelems&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(m_doc)&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(elementintersectsolidfilter)
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;oWrite1.WriteLine(<span style="color:#a31515;">&quot;intersecting&nbsp;elems:&nbsp;&quot;</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;intersectingelems.Count.ToString)
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;intersectingelem&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;<span style="color:blue;">In</span>&nbsp;intersectingelems
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;ElementParameters&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ParameterSet</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;intersectingelem.Parameters
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;param&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Parameter</span>&nbsp;<span style="color:blue;">In</span>&nbsp;ElementParameters
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;param.Definition.Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;OBJ-LOCATION-RDK&quot;</span>&nbsp;<span style="color:blue;">Then</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;<span style="color:blue;">Not</span>&nbsp;param.AsString&nbsp;=&nbsp;<span style="color:#a31515;">&quot;&quot;</span>&nbsp;<span style="color:blue;">Then</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;param.Set(param.AsString&nbsp;+&nbsp;<span style="color:#a31515;">&quot;+&quot;</span>&nbsp;+&nbsp;mass.Name)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;param.Set(mass.Name)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;populatedcounter&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MsgBox(<span style="color:#a31515;">&quot;Populated&nbsp;parameter&nbsp;OBJ-LOCATION-RDK&nbsp;for&nbsp;&quot;</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;populatedcounter.ToString&nbsp;+&nbsp;<span style="color:#a31515;">&quot;&nbsp;model&nbsp;elements&quot;</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Exit</span>&nbsp;<span style="color:blue;">For</span>
 
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Catch</span>&nbsp;ex&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Exception</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">Try</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MsgBox(<span style="color:#a31515;">&quot;No&nbsp;linked&nbsp;file&nbsp;by&nbsp;name&nbsp;of&nbsp;MassModel.rvt&nbsp;found&quot;</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
 
&nbsp;&nbsp;<span style="color:blue;">Else</span>
&nbsp;&nbsp;&nbsp;&nbsp;MsgBox(<span style="color:#a31515;">&quot;No&nbsp;linked&nbsp;files&nbsp;in&nbsp;project.&nbsp;&quot;</span>&nbsp;_
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:#a31515;">&quot;Please&nbsp;link&nbsp;a&nbsp;file&nbsp;named&nbsp;MassModel.rvt&nbsp;into&nbsp;project&quot;</span>)
&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
 
&nbsp;&nbsp;tx.Commit()
 
&nbsp;&nbsp;oWrite1.Close()
<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">Sub</span>
</pre>

The strange thing is this: it only returns the elements that have been offset or disjoined from their work plane.

Is there a simpler way to achieve what I am trying to do, like the `geometry.DoesIntersect` node in Dynamo? 


####<a name="3"></a>Coding Suggestions and Transformations

**Answer:** I'll begin with a comment or two on your sample code:

First, it is cleaner and safer and easier to encapsulate transactions in a `using` clause:
[using `using` automagically disposes and rolls back](http://thebuildingcoder.typepad.com/blog/2012/04/using-using-automagically-disposes-and-rolls-back.html).
Refer to The Building Coder topic group for more
on [handling transactions and transaction groups](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.53).

Secondly, you can access a parameter on an element directly by name.

It is better to use a display name independent method when possible, but it works well if you can guarantee that the given parameter name is unique on that element.

Then you can thus replace the following lines of code:

<pre class="code">
  Dim ElementParameters As ParameterSet = el.Parameters
  For Each elparam As Parameter In ElementParameters
    If elparam.Definition.Name = "OBJ-LOCATION-RDK" Then
      elparam.Set("")
      oWrite1.WriteLine(el.Category.Name)
    End If
  Next
</pre>

They can be replaced by something like:

<pre class="code">
  IList<Parameter> plist = e.GetParameters("OBJ-LOCATION-RDK")
  Parameter elparam = plist[0]
  elparam.Set("")
</pre>

That would be faster, more efficient, easier to read and understand.

It has nothing to do with your question, though, does it?

Looking closer at your specific question:

I was under the impression that all Dynamo code is public domain and open source, so you can explore the implementation of the `DoesIntersect` node yourself.

Have you checked out that possibility?

The mass element lives in `linkedfile.GetLinkDocument`, and so do the solids `potSol` that you extract from it. Is that correct?

The intersecting elements that you are searching for live in the project document `m_doc`, correct?

What is the transformation from the linked document into the project document?

Have you taken account of that?

It is interesting to hear that the intersection works in some cases, but not in others: 'elements offset or disjoined from their work plane'.

Maybe that is affecting their transformation in some way.

I recently discussed [filtering for intersecting elements](http://thebuildingcoder.typepad.com/blog/2018/03/create-2d-arc-and-filter-for-intersecting-elements.html) in
general.

Another, even more relevant discussion
concerns [linked file element intersection solid geometry](https://forums.autodesk.com/t5/revit-api-forum/linked-file-element-intersection-solid-geometry/m-p/7861611).
There, I make the following suggestion:

You have two solids, `A` in your main project `P` and `B` in your linked project `Q`.

- Determine the transformation `T` from the linked document `Q` coordinates to `P`'s.
- Open the linked project `Q` and retrieve the solid `Sb` of `B`.
- Transform it to `P`'s coordinate space: `T * Sb`.
- Retrieve the solid `Sa` of `A`.
- Calculate the intersection of `T*Sb` with `Sa`.
 
Now you can use an element intersection filter based on the intersection result like in the discussion mentioned before.

The transformation of `Sb` can be obtained using
the [SolidUtils.CreateTransformed method](http://www.revitapidocs.com/2018.1/22592761-f39c-4f53-d33b-6c21a4fa9d2d.htm).

Please check that out as well.


####<a name="4"></a>Solution by Applying Transformations

**Response:** You are correct; I had not considered the transformation of the linked mass elements, and it was only by dumb luck that every time I offset or disjoined a model element it would intersect with the default position of the linked masses.

Once the transform was added to the code it worked just as expected.

You are welcome to share this on your blog.

Here is the relevant code:

<pre class="code">
  <span style="color:green;">&#39;&nbsp;Determine&nbsp;transform&nbsp;of&nbsp;linked&nbsp;file&nbsp;in&nbsp;main&nbsp;project</span>
  
  <span style="color:blue;">Dim</span>&nbsp;transform&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Transform</span>&nbsp;=&nbsp;linkedfile.GetTotalTransform
   
  <span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;mass&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;<span style="color:blue;">In</span>&nbsp;massCollector
   
  &nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;massSolids&nbsp;<span style="color:blue;">As</span>&nbsp;Generic.<span style="color:#2b91af;">IEnumerable</span>(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">Solid</span>)&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;mass.Geometry(options).OfType(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">Solid</span>)()
   
  &nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;massSolid&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;<span style="color:blue;">In</span>&nbsp;massSolids
   
  &nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;(massSolid&nbsp;<span style="color:blue;">IsNot</span>&nbsp;<span style="color:blue;">Nothing</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">And</span>&nbsp;<span style="color:blue;">Not</span>&nbsp;massSolid.Edges.IsEmpty)&nbsp;<span style="color:blue;">Then</span>
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">&#39;&nbsp;Apply&nbsp;transform&nbsp;to&nbsp;mass</span>
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;transformedSolid&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Solid</span>&nbsp;=&nbsp;<span style="color:#2b91af;">SolidUtils</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.CreateTransformed(massSolid,&nbsp;transform)
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;elementIntersectsSolidFilter&nbsp;<span style="color:blue;">As</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementIntersectsSolidFilter</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementIntersectsSolidFilter</span>(transformedSolid)
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;intersectingElems&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(m_doc)&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsViewIndependent&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(elementIntersectsSolidFilter)
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">For</span>&nbsp;<span style="color:blue;">Each</span>&nbsp;intersectingElem&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Element</span>&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">In</span>&nbsp;intersectingElems
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;elParams&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">IList</span>(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">Parameter</span>)&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;intersectingElem.GetParameters(
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;Mass&nbsp;Intersections&quot;</span>)
   
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;elParams.Count&nbsp;&gt;&nbsp;0&nbsp;<span style="color:blue;">Then</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;elParam&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Parameter</span>&nbsp;=&nbsp;elParams(0)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">If</span>&nbsp;elParam.AsString&nbsp;=&nbsp;<span style="color:#a31515;">&quot;&quot;</span>&nbsp;<span style="color:blue;">Then</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;elParam.Set(mass.Name)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;populatedCounter&nbsp;+=&nbsp;1
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Else</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;elParam.Set(elParam.AsString&nbsp;_
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;<span style="color:#a31515;">&quot;+&quot;</span>&nbsp;+&nbsp;mass.Name)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">Next</span>
   
  &nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">If</span>
  &nbsp;&nbsp;<span style="color:blue;">Next</span>
  <span style="color:blue;">Next</span>
</pre>

Apart from cleaning it up a bit, the only changes made from the original case are in the lines beneath the comments.

<center>
<img src="img/ramboell2.png" alt="Rambøll" width="364"/>
</center>

Many thanks to Gustav for raising this interesting issue and sharing his solution!

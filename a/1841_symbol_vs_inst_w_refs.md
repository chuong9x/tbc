<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- REVIT-159855 [NewFamilyInstance ArgumentsInconsistentException line does not coincide with input face]
  https://forums.autodesk.com/t5/revit-api-forum/newfamilyinstance-face-line-familysymbol/m-p/9367327
  GetInstanceGeometry versus GetSymbolGeometry with references
  /a/case/sfdc/REVIT-159855/attach/
  Q: Whilst using the following NewFamilyInstance overloads:
  ItemFactoryBase.NewFamilyInstance(Reference, Line, FamilySymbol)
  ItemFactoryBase.NewFamilyInstance(Face, Line, FamilySymbol)
  I am getting the exception ArgumentsInconsistentException with message “Family cannot be placed on this line as it does not coincide with the input face.”
  I know that the line is coincident with the input face to a reasonable degree by comparison of coords and it is a curve resulting from the intersection of the input face with another face (so must be coincident with the input face).
  I believe the error is occurring because the test for compatibility is being conducted between a curve in instance space and a face is the symbol space of the originating geometry. I surmise this due to the fact that an uncut family instance (which inherently has solids transformed to instance space) throws the exception whilst a cut family instance (which inherently has solids in the instance position without transformation) does not.
  It can be further demonstrated that if the curve is transformed into symbol space then the exception is not thrown (since presumably comparison occurs between curve and face both in symbol space). However, when doing this the created hosted instance is not in a position that would be permissible in the UI i.e. it is in symbol space remote from the host element.
  Obviously if there is a better approach or I’m doing something wrong let me know. I don’t think I’m expected to add the instance in the wrong place and then move it to the right place etc. I believe this is an issue that has to be rectified in the API. Since it will currently work one way for walls, floors etc where solids are at the top level in the geometry tree but will fail for uncut family instances.
  A: The problem with the original code provided is the use of GeometryInstance.GetInstanceGeometry() when extracting solids from family instances.  What’s wrong with that?  Since this face will be used as a Reference for another instance placement, it is required to take the Reference from the faces/edges of that Solid.  GetInstanceGeometry() (as well as GetSymbolGeometry(Transform) and GetInstanceGeometry(Transform) ) have remarks that say:
  The geometry will be in the coordinate system of the model that owns this instance. The context of the instance object (such as effective material) will be applied to the symbol. Note that this method involves extensive parsing or Revit's data structures, so try to minimise calls if performance is critical. Geometry will be parsed with the same options as those used when this object was retrieved. This method returns a copy of the Revit geometry. It is suitable for use in a tool which extracts geometry to another format or carries out a geometric analysis; however, because it returns a copy the references found in the geometry objects contained in this element are not suitable for creating new Revit elements referencing the original element (for example, dimensioning). Only the geometry returned by GetSymbolGeometry() with no transform can be used for that purpose.
  This is basically the problem; the Picks (References in API) generated by Geometry parsing are not correct when we apply transforms and get the geometry in a modified form.  They can’t be used by routines that create elements from the input References (dimensions, tags, and other model elements creation).
  So, the fix is to make use of GetSymbolGeometry() to get the reference to the family instance face.  
  The complication lies in the logic for this particular workflow.  The line used for placement is calculated with the intersection of the instance face and the floor face.  So, we can’t get that line from the intersection of the symbol geometry and the floor face.  Thus, to make this work, I needed to also get the instance geometry for the family instance and use heuristics (comparing area and the transformed normal vector) to determine I’d found a matching face.  I believe in the next major release of Revit, we may be able to compare Tag values instead, though I didn’t test this.
  I also eliminated some of these steps for family instances which have their own unique geometry (by using the routine FamilyInstance.HasModifiedGeometry()).  This is probably better than trying to take that geometry through the full logic with transforms and inverse transforms when not necessary.
  Response: Thanks for the solution to this issue; it appears to be a very robust.  I've adjusted slightly to aid my understanding and updated to include non-family instances (since I need to find intersections between slabs and walls also).

twitter:

In-depth analysis of retrieving references from symbol versus instance geometry when placing an instance on a line in the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon https://bit.ly/refstosymbolinstance

Sooner or later, every serious Revit add-in developer will be scratching her head a bit over symbol versus instance geometry.
Here is a nice juicy challenge...
Wrap you head around this one in depth and future challenges in this area will seem trivial.
Retrieving references from symbol versus instance geometry when placing an instance on a line throws an <code>ArgumentsInconsistentException</code>...
 
&ndash; 
...

linkedin:

In-depth analysis of retrieving references from symbol versus instance geometry when placing an instance on a line in the #RevitAPI

https://bit.ly/refstosymbolinstance

Sooner or later, every serious Revit add-in developer will be scratching her head a bit over symbol versus instance geometry.
Here is a nice juicy challenge...
Wrap you head around this one in depth and future challenges in this area will seem trivial.
Retrieving references from symbol versus instance geometry when placing an instance on a line throws an ArgumentsInconsistentException...

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="600"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### References to Symbol versus Instance Geometry

Sooner or later, every serious Revit add-in developer will be scratching her head a bit over symbol versus instance geometry.

Here is a nice juicy challenge faced by Richard Thomas, one of the most knowledgeable and prolific solution authors in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160),
solved by Scott Conover, Engineering Director of the Revit development team.

Wrap you head around this one in depth and future challenges in this area will seem trivial:

The issue Richard faced was
a [NewFamilyInstance(Face,Line,FamilySymbol), ArgumentsInconsistentException](https://forums.autodesk.com/t5/revit-api-forum/newfamilyinstance-face-line-familysymbol/m-p/9367327):

**Question:** I am trying to use the following two `NewFamilyInstance` overloads:

<pre>
  ItemFactoryBase.NewFamilyInstance(Reference, Line, FamilySymbol)
  ItemFactoryBase.NewFamilyInstance(Face, Line, FamilySymbol)
</pre>

They throw the exception `ArgumentsInconsistentException` with the message, *Family cannot be placed on this line as it does not coincide with the input face.*

I know that the line is coincident with the input face to a reasonable degree by comparison of coords and it is a curve resulting from the intersection of the input face with another face (so must be coincident with the input face).

I believe the error is occurring because the test for compatibility is being conducted between a curve in instance space and a face is the symbol space of the originating geometry. I surmise this due to the fact that an uncut family instance (which inherently has solids transformed to instance space) throws the exception whilst a cut family instance (which inherently has solids in the instance position without transformation) does not.

It can be further demonstrated that if the curve is transformed into symbol space then the exception is not thrown (since presumably comparison occurs between curve and face both in symbol space). However, when doing this the created hosted instance is not in a position that would be permissible in the UI i.e. it is in symbol space remote from the host element.

Obviously if there is a better approach or I’m doing something wrong let me know. I don’t think I’m expected to add the instance in the wrong place and then move it to the right place etc. I believe this is an issue that has to be rectified in the API. Since it will currently work one way for walls, floors etc where solids are at the top level in the geometry tree but will fail for uncut family instances.

Here is a [zip file MinRepCase.zip](zip/rpt_MinRepCase.zip) providing a minimum reproducible case for this API behaviour.
There is a Revit project file within the solution called SampleProject.

**Later:** I’m not sure I can make any adjustments my end and if I did what would happen down the line? I can probably just avoid this overload and use a different form of family to achieve what I need. The idea was to add a shelf angle to the side of a beam/wall to pick up floor slab (bottom of floor intersects with web of beam or wall face). I can do this with level based structural framing and end offsets given the curve ends. Although it has the disadvantage of when the beam/wall is moved the angle is left behind. Ideally, I want to achieve the aforementioned hosted behaviour.

I don’t know if it has been concluded by the development team that this is acceptable internal behaviour or not i.e. can they suggest a workaround or correct alternative approach I should be using? Alternatively, do they believe the internal code needs adjustment and are planning to do so?

I think it seems straightforward in the sense that I get the curve by intersecting two faces. If I then supply one of those faces and the curve to the family instance constructor it shouldn’t tell me they don’t coincide. It’s a bit confusing otherwise and leads to contradictions in approach between elements with top-level solids and those with solids within the top-level geometry instance.

In the image below, I've drawn red lines in paintbrush to show where the instance is placed compared to where it should be placed (when we supply the curve transformed to symbol space). As can be seen, the element is hosted to the face of the beam. You would be hard pressed to achieve this using the UI (so as a minimum I should not be able to do this).

<center>
<img src="img/fi_placement.png" alt="Family instance placement" title="Family instance placement" width="800"/> <!-- 1150 -->
</center>

I note the view cube which shows that the orientation of the hosted family matches how the host family would be located in its family document (symbol space). I mention this because I've often used a transform from instance space to symbol space to identify the top face of the family (or other faces etc.). Regardless of the slope of the beam represented by the instance (where face normal is not 0,0,1) we can always transform it to symbol space where it will be. I note this because I believe it is a useful trick in identifying faces of instances with any orientation.

**Answer:** The problem with the original code provided is the use of GeometryInstance.GetInstanceGeometry() when extracting solids from family instances.

What’s wrong with that?

Since this face will be used as a Reference for another instance placement, it is required to take the Reference from the faces/edges of that Solid.
[GetInstanceGeometry()](https://www.revitapidocs.com/2020/22d4a5d4-dfc2-7227-2cae-b989729696ec.htm) (as well as
[GetSymbolGeometry(Transform)](https://www.revitapidocs.com/2020/6de9b5fd-682f-ffa0-5e49-84b1d227d606.htm) and
[GetInstanceGeometry(Transform)](https://www.revitapidocs.com/2020/d5aad2b5-3211-3800-9f1e-1af921e73902.htm))
have remarks that say:

> The geometry will be in the coordinate system of the model that owns this instance. The context of the instance object (such as effective material) will be applied to the symbol. Note that this method involves extensive parsing or Revit's data structures, so try to minimise calls if performance is critical. Geometry will be parsed with the same options as those used when this object was retrieved. This method returns a copy of the Revit geometry. It is suitable for use in a tool which extracts geometry to another format or carries out a geometric analysis; however, because it returns a copy the references found in the geometry objects contained in this element are not suitable for creating new Revit elements referencing the original element (for example, dimensioning). Only the geometry returned by GetSymbolGeometry() with no transform can be used for that purpose.

This is basically the problem; the Picks (References in API) generated by Geometry parsing are not correct when we apply transforms and get the geometry in a modified form.  They can’t be used by routines that create elements from the input References (dimensions, tags, and other model elements creation).

So, the fix is to make use of `GetSymbolGeometry` to get the reference to the family instance face.  

The complication lies in the logic for this particular workflow.  The line used for placement is calculated with the intersection of the instance face and the floor face.  So, we can’t get that line from the intersection of the symbol geometry and the floor face.  Thus, to make this work, I needed to also get the instance geometry for the family instance and use heuristics (comparing area and the transformed normal vector) to determine I’d found a matching face.  I believe in the next major release of Revit, we may be able to compare Tag values instead, though I didn’t test this.

I also eliminated some of these steps for family instances which have their own unique geometry (by using the routine FamilyInstance.HasModifiedGeometry()).  This is probably better than trying to take that geometry through the full logic with transforms and inverse transforms when not necessary.

Here is a link to the modified VB code achieving this:

- [MinRepCase_modified.vb](zip/rpt_MinRepCase_modified_vb.txt)

**Response:** Thanks for the solution to this issue; it appears to be a very robust.

I've adjusted slightly to aid my understanding and updated to include non-family instances (since I need to find intersections between slabs and walls also).

- [MinRepCase_modified2.vb](zip/rpt_MinRepCase_modified2_vb.txt)

Many thanks to Richard for raising this tricky issue and to Scott for the illuminating solution!
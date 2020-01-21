<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</head>

<!---

- Ahmed Nabil suggested a neat method to detect when a specific command is launched by checking for its name in the DocumentChanged event:
  Group Edit Event
  https://forums.autodesk.com/t5/revit-api-forum/group-edit-event/m-p/8169237

- SelectableInViewFilter
  https://www.revitapidocs.com/2018.2/4def5498-f47f-870c-ea25-0408b6603dac.htm

- access project location in forge
  [Q] I am working with Forge and I have a JSON file of the models, but there is no Location information inside that file. I can only access the project information’s address, but that isn’t what I want. 
  Is there a way to get the Location info inside the JSON file during the conversion? I mean, the location info you have in Revit at:
  Manage > Project location section > Location
  [A] I can only suggest that you explore the Forge JSON and SVF data further and search deeper.
  If the project information is not present there, I would suggest that you export it from the RVT using the Revit API.
  You can do so either on the desktop in an installed Revit session, or in the Forge environment using the Design Automation API.

twitter:

 in the #RevitAPI #DynamoBim @AutodeskForge @AutodeskRevit #bim #ForgeDevCon

&ndash; 
...

linkedin:


#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<p style="font-size: 80%; font-style:italic"></p>

-->

### Torsion Tools and Command Event


#### <a name="2"></a>Torsion Tools GitHub Launch and Solution Overview

An exciting [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
announces the [Torsion Tools GitHub launch and basic solution overview](https://forums.autodesk.com/t5/revit-api-forum/torsion-tools-github-launch-and-basic-solution-overview/m-p/9254509):

> I wanted to share my Autodesk Revit 2020 API Visual Studio Solution Template with Code Examples for Common Tools.

> The intent of the repository is to help provide Revit API support and to share code examples and solutions for common issues or time consuming tasks in Revit.
I will provide examples and tools of items as I have created and use them, and then other are free to manipulate them to fit how they need them to work, but without any expressed or implied warranty and/or guarantee of functionality or errors.
I think this will be the best way to share some of the work I have done without having to create installers for every single tool or publishing everything to the Autodesk App Store.

- [TorsionTools GitHub repository](https://github.com/TorsionTools/R20)
- [Six-minute video recording on YouTube](https://youtu.be/3EVx9SzKJbk)

<iframe width="480" height="270" src="https://www.youtube.com/embed/3EVx9SzKJbk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Many thanks to Torsion Tools for sharing and documenting this!


#### <a name="3"></a>Detect Command Launch

[Ahmed Nabil](https://www.linkedin.com/in/anabil1) suggested
a neat method to detect when a specific command is launched by checking for its name in the `DocumentChanged` event, in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Group Edit Event](https://forums.autodesk.com/t5/revit-api-forum/group-edit-event/m-p/8169237):

**Question:** Is it possible somehow to know when a user enters the group edit mode? 

**Answer:** You can do this by handling the `DocumentChanged` event and checking if the transaction name is "Edit Group":

<pre class="code">
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Result</span>&nbsp;OnStartup(&nbsp;<span style="color:#2b91af;">UIControlledApplication</span>&nbsp;a&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;a.ControlledApplication.DocumentChanged
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+=&nbsp;ControlledApplication_DocumentChanged;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Result</span>.Succeeded;
&nbsp;&nbsp;}
 
&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ControlledApplication_DocumentChanged(
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">object</span>&nbsp;sender,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">DocumentChangedEventArgs</span>&nbsp;e&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;txname&nbsp;=&nbsp;e.GetTransactionNames()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.FirstOrDefault();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;txname&nbsp;==&nbsp;<span style="color:#a31515;">&quot;Edit&nbsp;Group&quot;</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;user&nbsp;enters&nbsp;Edit&nbsp;Group&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
</pre>

This event is fired both on entering the Edit Mode (under the transaction name "Edit Group") and on exiting (under the transaction name "Finish Edit Group").

Thank you, Ahmed, for the nice solution!


#### <a name="4"></a>SelectableInViewFilter

I never before noticed, much less used, the [SelectableInViewFilter](https://www.revitapidocs.com/2018.2/4def5498-f47f-870c-ea25-0408b6603dac.htm):

> A filter that passes elements that are selectable in the given view.

> This filter is a slow filter... This filter is designed to operate on a list of elements visible in the given view. This can be obtained from a `FilteredElementCollector` constructed with the view id. This filter may not correctly restrict elements which are not a part of the visible elements of the view.

Sounds pretty interesting, and useful...

Do you happen to have any interwsting examples?

#### <a name="5"></a>Access Project Location in Forge

**Question:** I am working with Forge and I have a JSON file of the model, but there is no Location information inside that file.
I can only access the project information’s address, but that isn’t what I want. 
Is there a way to get the Location info inside the JSON file during the conversion?
I mean, the location info you have in Revit at:

-  Manage &rarr; Project location section &rarr; Location

**Answer:** I can only suggest that you explore the Forge JSON and SVF data further and search deeper.
If the project information is not present there, I would suggest that you export it from the RVT using the Revit API.
You can do so either on the desktop in an installed Revit session, or in the Forge environment using the Design Automation API.

#### <a name="6"></a>Distinguish Structural and Access Element Volume Information in Forge

**Question:** I heard that one way of being able to distinguish structural elements in a model authored by Revit was that they would possess a volume property.
 
However, I have encountered Revit models that do not contain a volume property on elements that should have one.
They are indeed structural elements. Have you encountered this problem? Could it be caused by a problem in Revit, or is it more likely to be an error when the client enters the data for the model? Is there any other way we could determine a structural element?
 
The second issue I have is that I inevitably have to support models that are authored by other programs that do not provide a volume property. I imagine it will be very difficult, if not impossible, to find a solution that can cover all possible authoring programs. One thing I was hoping that might be a workaround is to use the Design Automation API to process "other" models with Revit. Would this be possible? This would only be a solution provided we can solve the first issue of reliably distinguishing structural elements.
 
**Answer:** You can easily and reliably distinguishing structural elements in a Revit model by simply checking their `Category` property.

If a CAD model produced by a third party can be imported properly into Revit so that structural elements are correctly recognised, they will also be equipped with the appropriate category.

I would definitely not try to use the volume property to distinguishing structural elements.
All BIM elements that have a volume can be equipped with a property reporting its value.
That property is in no way whatsoever specific to structural elements.

If you really are interested in the volume of an element for other reasons, please read on. Else, you can skip the following:

Many Revit elements may include a pre-calculated volume property. Its value may be unreliable. In the Revit API, I would recommend calculating the volume of an element by querying the element solid for its volume, not using any data stored in the pre-calculated element properties.
 
However, the main part of the question is on how to achieve this with non-Revit models.
I see no problem, and only one obvious answer: grab the geometry from the model in any format you can get, assembly solids from those, and query their volume. If you can import the models in Revit, then it is fine, otherwise of course you would require a robust solid modeler library to achieve this.
 
Retrieving the solid geometry from an element and querying its volume is an operation that the Forge Design Automation for Revit will support.
Several discussions by The Building Coder deal with determining areas and volumes of BIM elements using the desktop Revit API, e.g.:

- [Calculating gross and net wall areas](https://thebuildingcoder.typepad.com/blog/2015/03/calculating-gross-and-net-wall-areas.html)
- [Gross and net wall area calculation enhancement](https://thebuildingcoder.typepad.com/blog/2015/04/gross-and-net-wall-area-calculation-enhancement-and-events.html)
 
The same processing can also be performed in [Forge Design Automation for Revit (DA4R)](https://thebuildingcoder.typepad.com/blog/about-the-author.html#5.55).



<center>
<img src="img/.png" alt="" title="" width="100"/> <!-- 604 -->
</center>


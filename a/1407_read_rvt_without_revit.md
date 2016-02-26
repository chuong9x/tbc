<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="run_prettify.js" type="text/javascript"></script>
<!---
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---

11545374 [Import data from rvt file - similar to RealDWG]

#dotnet #csharp
#fsharp #python
#grevit
#responsivedesign #typepad
#ah8 #augi #dotnet
#stingray #rendering
#3dweb #3dviewAPI #html5 #threejs #webgl #3d #mobile #vr #ecommerce
#Markdown #Fusion360 #Fusion360Hackathon
#javascript
#RestSharp #restAPI
#mongoosejs #mongodb #nodejs
#rtceur
#xaml
#3dweb #a360 #3dwebaccel #webgl @adskForge
@AutodeskReCap @Adsk3dsMax
#revitAPI #bim #aec #3dwebcoder #adsk #adskdevnetwrk @jimquanci @keanw
#au2015 #rtceur
#eraofconnection
#RMS @researchdigisus
@adskForge #3dwebaccel
#a360

Revit API, Jeremy Tammik, akn_include

Reading an RVT File without Revit #revitAPI #3dwebcoder @AutodeskRevit #bim #aec #adsk #adskdevnetwrk

A huge number of developers have requested a possibility to read or even write an RVT file without running Revit, similar to the possibility to read and write a DWG file without AutoCAD installed. For the latter, one option that has been around for ages is to make use of the RealDWG library for reading and writing DWG files. A more recent and future oriented option is provided by the cloud-based AutoCAD I/O web service. Here is yet another variation on this query &ndash; Question: Is there any way that I can read a Revit model without having Revit installed on the users machine? ...

-->

### Reading an RVT File without Revit

A huge number of developers have requested a possibility to read or even write an RVT file without running Revit, similar to the possibility to read and write a DWG file without AutoCAD installed.

For the latter, one option that has been around for ages is to make use of the [RealDWG](http://www.autodesk.com/realdwg) library for reading and writing DWG files.

A more recent and future oriented option is provided by the cloud-based [AutoCAD I/O](http://autocad.io) web service.

Here is yet another variation on this query:

**Question:** Is there any way that I can read a Revit model without having Revit installed on the users machine?

I'm thinking something similar to RealDWG for AutoCAD?

The closest I was able to find is your discussion
of [Open Revit OLE Storage](http://thebuildingcoder.typepad.com/blog/2010/06/open-revit-ole-storage.html).

However, I got the impression that this provides very limited and that it would not enable to extract, say, specific wall properties.

**Answer:** Yes, the method that you refer to is extremely limited in the scope of information that can be accessed.

It is based on OLE structured storage, COM Structured Storage and the Compound File Binary Format.

One additional aspect of the structure storage format is that it also enables access to
so-called [custom file properties on RVT and RFA files](http://thebuildingcoder.typepad.com/blog/2015/09/lunar-eclipse-and-custom-file-properties.html#3),
since they are standard Windows properties stored there.

Here is another recent short overview
on [programmatically generating Revit family RFA files](http://thebuildingcoder.typepad.com/blog/2015/06/getting-started-creating-families-and-rfa-files.html#4).

There is a little bit more data that can indeed be accessed from family definition RFA files without running Revit using the `Application` `ExtractPartAtomFromFamilyFile` method:

<!--- 0248 0448 -->

- [Extract Part Atoms](http://thebuildingcoder.typepad.com/blog/2009/11/extract-part-atoms.html)
- [Extract Part Atom Revisited](http://thebuildingcoder.typepad.com/blog/2010/09/extract-part-atom-revisited.html)

Regarding the question you raise on full access to the RVT file contents without a Revit session up and running, we recently published a request for wishes concerning that area:

<center>
[What Can Revit on the Cloud Do For You?](http://thebuildingcoder.typepad.com/blog/2016/02/what-can-revit-on-the-cloud-do-for-you.html#2)
</center>

If you have a serious need and a clear vision of what you would like to see, please contact [Jim Quanci](mailto:jim.quanci@autodesk.com) and let him know.

It will be much appreciated.

Thank you!


Oh, and before signing off, here is another thought-provoking idea:

#### <a name="2"></a>A House for $20000 &ndash; but it's Nicer Than Yours

Check out this [Rural Studio](http://www.ruralstudio.org) architectural project to
build [a house that costs just $20,000 &ndash; but it’s nicer than yours](http://www.fastcoexist.com/3056129/this-house-costs-just-20000-but-its-nicer-than-yours/1).

<center>
<img src="img/20k_house.jpg" alt="20k house" width="400">
</center>

Look at their list of [beautiful low-budget 5th year architecture projects](http://www.ruralstudio.org/projects.html).

I bet they made many people very happy with those!
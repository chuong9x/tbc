<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
</head>

<!---

- Get Revit Camera FOV
  Many people have asked how you can retrieve the Revit camera field of view or FOV.
  As Valerii Nozdrenkov points out in
  his [comment](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html#comment-4891620499)
  on [mapping the Revit camera settings to the Forge viewer](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html),
  The Building Coder already published a solution suggested by Arno&scaron;t L&ouml;bel reading the required data from
  the [custom exporter `GetCameraInfo`](https://thebuildingcoder.typepad.com/blog/2014/09/custom-exporter-getcamerainfo.html):
  > When a view is processed and run through a custom exporter context, its properties are used to populate a `ViewNode` instance.
  > One of its methods is `GetCameraInfo`, which provides information that ought to cover everything you need to know about the view's camera.
  search for and answer other question on this on the web...
  
- Would you like to work with the Forge development team?
We hope you’re all staying safe and healthy in your homes.
Autodesk is still actively hiring. 
Forge continues to search for top talent as our global workforce works remotely.
Would you be a good fit? 
Here are a few roles highlighted in April and May:
- [20WD39053 &ndash; Senior Technical Writer, Developer Content &ndash; This role can be located anywhere](https://rolp.co/68d5i)
- [20WD39597 &ndash; Senior Software Engineer, Cloud Service &ndash; Shanghai](https://rolp.co/IUdfi)
- [19WD36553 &ndash; Principal Engineer &ndash; UK](https://rolp.co/Pc5Li)
- [20WD38369 &ndash; Software Architect &ndash; Singapore](https://rolp.co/JWHti)
- [20WD39627 &ndash; Senior Vendor Manager &ndash; San Francisco](https://rolp.co/RUAfi)
- [20WD38632 &ndash; Engineering Manager &ndash; Portland](https://rolp.co/Dmtei)
- [20WD38618 &ndash; Software Development Manager &ndash; Novi](https://rolp.co/SrG6i)


twitter:

the #RevitAPI @AutodeskForge @AutodeskRevit #bim #DynamoBim #ForgeDevCon

&ndash; 
...

linkedin:

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="600"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### Revit Camera FOV, Forge Partner Talks and Jobs

Today we resuscitate a five-year old Revit API answer, still as fresh and useful as ever, followed by Forge and job opportunities truly fresh off the presses:

- [Determining the Revit Camera FOV](#2)
- [Forge Partner Talks](#3)
- [Jobs at Autodesk](#4)

####<a name="2"></a> Determining the Revit Camera FOV

Many people have asked how you can retrieve the Revit camera field of view or FOV.

As Valerii Nozdrenkov points out in
his [comment](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html#comment-4891620499)
on [mapping the Revit camera settings to the Forge viewer](https://thebuildingcoder.typepad.com/blog/2019/06/revit-camera-settings-project-plasma-da4r-and-ai.html),
The Building Coder already published a solution suggested by Arno&scaron;t L&ouml;bel reading the required data from
the [custom exporter `GetCameraInfo`](https://thebuildingcoder.typepad.com/blog/2014/09/custom-exporter-getcamerainfo.html):

> When a view is processed and run through a custom exporter context, its properties are used to populate a `ViewNode` instance.

> One of its methods is `GetCameraInfo`, which provides information that ought to cover everything you need to know about the view's camera.

<pre class="code">
  public RenderNodeAction OnViewBegin(ViewNode node)
  {
    CameraInfo cameraInfo = node.GetCameraInfo();
    
    var view = document.GetElement(node.ViewId);
    return RenderNodeAction.Proceed;
  }
</pre>

It seems worthwhile to reiterate this, since the question keeps popping up...

<center>
<!--
<img src="img/camera_fov_focal_length_distance_animation_480.gif" alt="Camera focal length" title="Camera focal length" width="240"/>
<p style="font-size: 80%; font-style:italic">[By SharkD](http://commons.wikimedia.org/wiki/User:SharkD), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) &ndash; In this simulation, adjusting the angle of view and distance of the camera while keeping the object in frame results in vastly differing images. At distances approaching infinity, the light rays are nearly parallel to each other, resulting in a 'flattened' image. At low distances and high angles of view objects appear 'foreshortened'.</p>
-->

<img src="img/camera_fov_lens_angle_of_view.png" alt="Camera angle of view" title="Camera angle of view" width="249"/>

</center>


####<a name="3"></a> Forge Partner Talks

The virtual Forge accelerators are leading to a large number of technical webinars and zoom meetings with great attendance and participation.

It is time to also hold some 'business creation' meetings with Forge partners.
 
Here are some upcoming webinars in this area that might be of interest to you:

- [Forge Partner Talks: Digital Twins](https://autodesk.zoom.us/webinar/register/7415875742427/WN_UiMEtQNiTFiPH8T_ekwk4w) (May 6 @ 7am PST/4pm CET)
&ndash; Digital twins were one of the top ten tech trends last year, according to Gartner and Deloitte. Come and hear from Forge partners who have built or leveraged successful digital twin solutions and see how you can too.
- [Forge Partner Talks: AR/VR](https://autodesk.zoom.us/webinar/register/6515877525566/WN_W6abt1RSR5K3V44HV5CtXQ) (May 13 @ 7am PST/4pm CET)
&ndash; See how leading Autodesk partners are bringing Forge-powered AR/VR solutions to the design and make space and learn how these solutions can benefit you.
- [Forge Partner Talks: Configurators](https://autodesk.zoom.us/webinar/register/8415877525897/WN_01GeR_H9RKOGTOg67Unagg) (May 20 @ 7am PST/4pm CET)
&ndash; Hear from Forge partners who have increased their customer satisfaction by creating Forge-powered configurators and see how providing similar customization tools to your customers can help you maintain your competitive edge.
- [Forge Partner Talks: Dashboards and Insights](https://autodesk.zoom.us/webinar/register/7515877526195/WN_vDxsTlF4QgS3s_8qIXYEXg) (May 27@  7am PST/4pm CET) 
&ndash; It's all about the data! Discover some of the innovative dashboards our top Forge partners have built, and see how these types of solutions can help you better visualize your data, draw insights, and be more profitable.

Looking forward to having you there!
 
 
####<a name="4"></a> Jobs at Autodesk

Finally... Would you like to work with the Forge development team?

We hope you’re all staying safe and healthy in your homes.

In spite of the current situation, Autodesk is still actively hiring.

Forge continues to search for top talent as our global workforce works remotely.

Would you be a good fit? 

Here are a few roles highlighted in April and May:

- [20WD39053 &ndash; Senior Technical Writer, Developer Content &ndash; This role can be located anywhere](https://rolp.co/68d5i)
- [20WD39597 &ndash; Senior Software Engineer, Cloud Service &ndash; Shanghai](https://rolp.co/IUdfi)
- [19WD36553 &ndash; Principal Engineer &ndash; UK](https://rolp.co/Pc5Li)
- [20WD38369 &ndash; Software Architect &ndash; Singapore](https://rolp.co/JWHti)
- [20WD39627 &ndash; Senior Vendor Manager &ndash; San Francisco](https://rolp.co/RUAfi)
- [20WD38632 &ndash; Engineering Manager &ndash; Portland](https://rolp.co/Dmtei)
- [20WD38618 &ndash; Software Development Manager &ndash; Novi](https://rolp.co/SrG6i)


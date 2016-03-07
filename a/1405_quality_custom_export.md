<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="run_prettify.js" type="text/javascript"></script>
<!---
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---

- i survived the reorg that
  [kean mentioned](http://through-the-interface.typepad.com/through_the_interface/2016/02/time-for-a-change.html).
  [Securing your AutoCAD app using .NET](http://through-the-interface.typepad.com/through_the_interface/2016/02/securing-your-autocad-app-using-net.html)
  One of my closer colleagues had to leave, though, Vladimir of the Moscow office.

- publish jim's talk on 1_future_of_making_things_iot_forge
  /a/doc/hack/2016-01-26_bim_programming_madrid/1_future_of_making_things_iot_forge.pdf
  clean up 1_future_of_making_things_iot_forge_public.pptx slides
  and integrate into 1_future_of_making_things_iot_forge.rtf
  /a/doc/hack/2016-01-26_bim_programming_madrid/1_future_of_making_things_iot_forge.rtf

- forge devcon -- Caroline Ward FW: Autodesk® Developer News Special Edition - February 16, 2016

- https://www.linkedin.com/pulse/top-12-things-i-learned-ted-2016-bill-gross
  the last three items are of special interest to developers:
  10. desire paths and low friction in design
  11. Deep Learning and a [self-driving-car](http://www.bloomberg.com/features/2015-george-hotz-self-driving-car)
  12. virtual reality (VR) and augmented reality (AR)

- exporting data:
  FireRating
  Adn Xtra Lab?
  ArchSample
  RoomSchedule
  RDBLink_2010.zip
  cloud based FireRating sample

- [Revit QRCode Generator](https://www.linkedin.com/pulse/revit-qrcode-generator-fausto-mendez) by
  Fausto Mendez, Technology Manager at Corbins Electric
  http://www.faustomendez.com/2016/02/revit-qrcode-generator/
  utilize the power of a cloud base API for the QR Code generation  and System.Net to retrieve that generated image from the cloud.
  to generate a QR code image with an embedded link that points to the material specs website page using the URL link that is located inside this family’s Type Properties.
  the QR Code image can be used in a Label Schedule, Title block or inside a detail sheet:
  fm_ScheduleQrcode-1-1024x527.png
  Screen shoot of the scanned QR Code on a smartphone:
  fm_qrc-768x1365.png

- 11377787 [Quality of the geometry on Custom Export]
  The level of detail of tessellated surfaces is controlled via LevelOfDetail property of ViewNode of which instance is received in OnViewBegin method of the IExportContext interface.
  The level of detail can only be set for the entire view, not for individual elements. It is so because alternating LoD between element would create discrepancies in tessellated meshes.


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

Revit API, Jeremy Tammik, akn_include

Reorg, FoMT, DevCon, TED, QR, Custom Exporter Quality #revitAPI #3dwebcoder @AutodeskRevit @adskForge #3dwebaccel #a360 #bim

I had last week off and terminated it with a nice ski tour climbing the Albristhorn.
Here are some topics to get going again
&ndash; Reorg and Octo
&ndash; The Future of Making Things, Iot, Forge and more
&ndash; Forge and the DevCon in SF
&ndash; T<sup>4</sup> &ndash; Top Twelve TED Topics
&ndash; RDBLink and Exporting data from Revit
&ndash; Revit QRCode generator
&ndash; Controlling custom exporter geometry quality...

-->

### Reorg, FoMT, DevCon, TED, QR, Custom Exporter Quality

I had last week off and terminated it with a nice ski tour climbing the Albristhorn:

<center>
<a data-flickr-embed="true"  href="https://www.flickr.com/photos/jeremytammik/albums/72157664235570149" title="Skitour Albristhore"><img src="https://farm2.staticflickr.com/1624/25056609062_66b1e71786_n.jpg" width="320" height="240" alt="Skitour Albristhore"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
</center>

Here are some topics to get going again:

- [Reorg and Octo](#2)
- [The Future of Making Things, Iot, Forge and more](#3)
- [Forge and the DevCon in SF](#4)
- [T<sup>4</sup> &ndash; Top Twelve TED Topics](#5)
- [RDBLink and Exporting data from Revit](#6)
- [Revit QRCode generator](#7)
- [Controlling custom exporter geometry quality](#8)


#### <a name="2"></a>Reorg and Octo

[Autodesk restructured](http://news.autodesk.com/press-release/corporate-sustainability/autodesk-announces-restructuring-plan-accelerate-transition-c) the
week before I left.

Luckily, I survived that.

My colleague Kean Walmsley only escaped
by [switching to a different team](http://through-the-interface.typepad.com/through_the_interface/2016/02/time-for-a-change.html),
OCTO, the Office of the CTO, which is a very nice career move, in fact.

Congratulations on that, Kean!

The changes did affect several others of my closer colleagues, though.

Good luck to them in their new projects!

A lot of Kean's AutoCAD.NET discussions are also valid for Revit, by the way, such as these recent ones:

- [Disabling the AutoCAD ribbon using .NET](http://through-the-interface.typepad.com/through_the_interface/2016/02/disabling-the-autocad-ribbon-using-net.html)
- [Disabling AutoCAD’s complete UI using .NET](http://through-the-interface.typepad.com/through_the_interface/2016/02/disabling-autocads-complete-ui-using-net.html)
- [Securing your AutoCAD app using .NET](http://through-the-interface.typepad.com/through_the_interface/2016/02/securing-your-autocad-app-using-net.html)

The latter one highlights the Entitlement API, which was also covered in the past specifically for Revit and Exchange Store Apps by Mikako Harada and Daniel Du, who now also left the company:

- [Entitlement API for Revit Exchange Store Apps](http://adndevblog.typepad.com/aec/2015/04/entitlement-api-for-revit-exchange-store-apps.html)
- [Exchange Apps resources &ndash; Entitlement API information](http://thebuildingcoder.typepad.com/blog/2014/05/exchange-apps-webinar-recording-and-resources.html#3)

Talking about restructuring and so on:


#### <a name="3"></a>The Future of Making Things, Iot, Forge and More

Restructuring and reorienting to take advantage of new opportunities and technologies is not just something we face within Autodesk &ndash; of course all other fellow developers do so too, including, probably (hopefully!), you.

In order to get a handle on the Autodesk view of things and a number of important hints for the developer community, I published my notes from Jim Quanci's presentation
on [The Future of Making Things, Iot, Forge and More](http://the3dwebcoder.typepad.com/blog/2016/02/future-of-making-things-iot-forge-devcon.html#3).

Jim presents an exciting, visionary, yet down-to earth and practical guide for the developer community today.

It was used as the introductory keynote presentation for all of our
recent [DevDay conferences](http://autodeskdevdays.com)
and the recent four-day [Autodesk Cloud Accelerator](http://autodeskcloudaccelerator.com) in Munich.

The main topic is how design, engineering, making and operating are changing &ndash; changes that effect all of us &ndash; the tool makers.

I like it so much that I also showed it at
the [BIM Programming](http://www.bimprogramming.com) conference in Madrid.

I hope you find it interesting too!



#### <a name="4"></a>Forge and the DevCon in SF

Carrying on from the future of making things, let's look at [Forge](http://forge.autodesk.com).

The [Forge DevCon](http://forge.autodesk.com/conference) developer conference registration being
held at Fort Mason in San Francisco on June 15-16, 2016 is now open.

Forge and the conference address the technologies being used to create the Future of Making Things, such as Virtual and Augmented Reality, BIM, Computational Design, ERP, PLM, Internet of Things, 3D Printing, Digital Fabrication, Fusion 360, web-based 2D and 3D design visualization, etc.

You can save a lot of money be registering and booking your hotel early.

You might also want to come early for the conference, stay late, e.g., to take advantage of the opportunity to participate in cloud and web technology Meetups on June 13-14 that can only be found in Northern California and to participate in the Computational Design & Digital Fabrication or
the [Forge Cloud Accelerator](http://autodeskcloudaccelerator.com) at
Autodesk June 20-24.

Learn all about [Forge DevCon and why to attend](http://forge.autodesk.com/conference),
and get your early bird discount by [booking your ticket here](https://forgedevcon.eventbrite.com).

If you have any questions, please reach out to us
at (mailto:contact.forgedevcon@autodesk.com).

Forge is Autodesk’s Cloud based design and engineering app platform of the future (and now) connecting people that Design, Make and Use. Visit [forge.autodesk.com](http://forge.autodesk.com) for more information.




#### <a name="5"></a>T<sup>4</sup> &ndash; Top Twelve TED Topics

While we are on the topic of future oriented technologies, you should not miss this personal selection by Bill Gross of his [top twelve things learned at TED 2016](https://www.linkedin.com/pulse/top-12-things-i-learned-ted-2016-bill-gross).

The last three items are of special interest to us designers and developers:

- 10 &ndash; Desire Paths and Low Friction in Design
- 11 &ndash; Deep Learning and a [self-driving-car](http://www.bloomberg.com/features/2015-george-hotz-self-driving-car)
- 12 &ndash; Virtual Reality (VR) and Augmented Reality (AR)


#### <a name="6"></a>RDBLink and Exporting data from Revit

Now for some pure Revit API oriented topics as well.

An obvious and frequently asked question is how to export data from Revit.

It came up again, prompted by an update to the ancient Revit API discussion forum thread
on [Revit Server REST API and Project Data Exporting](http://forums.autodesk.com/t5/revit-api/revit-server-rest-api-and-project-data-exporting/m-p/3810739) and
a new thread
on [support in exporting database](http://forums.autodesk.com/t5/revit-api/need-support-in-exporting-database/m-p/5602871):

**Question:** Is there a way to get all information regarding a model or project in a transferable format?

I have heard of ODBC links and DBLink, but I'm looking for an API enabled way to get data and then push it into a secondary system.

JSON, CSV, XML I can work with any format and do any of the necessary scrubbing and ETL leg work.

**Question 2:** I want to create a desktop application that can export RVT data to a SQLServer.

**Question 3:** We are building a plugin in C# using API that exports project data to our SQL server. DBLink has been good so far but now we need to enhance and make it feature rich for our users.

Can you provide source code for DBLink please?  If it's in open source, or guide us in right direction. Or some method in which we are able to transfer all the project data direct to our SQL server.

Basically what we are looking is a simple UI that users can export Revit project data into our SQL server; we are making certain reports based on this export.

**Answer:** First of all, good luck with this. It sounds like you've got quite a lot of work ahead of you.

I'd recommend taking a look at the source code for RevitLookup.  It provides a good starting point for figuring out how to access the data for different kinds of elements.  Unfortunately, you'll notice is that not all model data is accessible through the API.

The other addin you'll want to take a look at is the Revit DB Link addin.  It's also from Autodesk and available to subscription customers for free.  To automate the export, you may be able to use some sort of UI Automation.  It may be able to export more data than what you'd normally be able to access through the API, but I suspect it would have the same limitations.

You may also want to take a look at programs like Codebook and Ideate BIMLink, which have some pretty powerful data import/export capabilities.

Here are some other relevant SDK samples and topics on The Building Coder:

- FireRating SDK sample
- Some of the [Adn Xtra Labs](https://github.com/jeremytammik/AdnRevitApiLabsXtra)
- ArchSample SDK sample
- RoomSchedule SDK sample
- The [FireRatingCloud](https://github.com/jeremytammik/FireRatingCloud) cloud based FireRating sample
- The entire [Export category](http://thebuildingcoder.typepad.com/blog/export)

Returning to the RDBLink sample mentioned above, its source code was originally provided as part of the Revit SDK, before it was promoted to an official subscription package.

Here is the [last Revit SDK version of RDBLink](zip/RDBLink_2010.zip) for Revit 2010.

I have no idea how useful it might be to migrate it to Revit 2016.

It will show you all the principles, at least.

I find it much easier and more effective to export to a cloud-based NoSQL database rather than SQL, nowadays, though.

More on this anon.

Finally, here are some pretty ancient articles discussing various uses of RDBLink:

- [Integration with a Database or ERP System](http://thebuildingcoder.typepad.com/blog/2009/07/integration-with-a-database-or-erp-system.html)
- [Adding a Column to RDBLink Export](http://thebuildingcoder.typepad.com/blog/2009/11/adding-a-column-to-rdblink-export.html)
- [Retrieving Project Parameters](http://thebuildingcoder.typepad.com/blog/2010/01/retrieving-project-parameters.html)
- [Parameter Access and Scheduling](http://thebuildingcoder.typepad.com/blog/2010/05/parameter-access-and-scheduling.html)
- [Exporting Parameter Data to Excel, and Re-importing](http://thebuildingcoder.typepad.com/blog/2012/09/exporting-parameter-data-to-excel.html)
- [Survey and Project Base Point](http://thebuildingcoder.typepad.com/blog/2012/11/survey-and-project-base-point.html)


#### <a name="7"></a>Revit QRCode Generator

Here is a really easy one:

I'll simply point out this nice blog post by Fausto Mendez, Technology Manager at Corbins Electric, who presents a full implementation of
a [Revit QRCode Generator](http://www.faustomendez.com/2016/02/revit-qrcode-generator)
([^](https://www.linkedin.com/pulse/revit-qrcode-generator-fausto-mendez)).

In Fausto's own words, it utilises the power of a cloud based API for the QR Code generation and System.Net to retrieve the result to generate a QR code image with an embedded link that points to the material specs website page using the URL link that is located inside this family’s Type Properties.

The QR Code image can be used in a Label Schedule, Title block or inside a detail sheet:

<center>
<img src="img/fm_ScheduleQrcode-1-1024x527.png" alt="QR Code image used in a Label Schedule" width="500">
</center>

Here is a screen snapshot of the scanned QR code on a smartphone:

<center>
<img src="img/fm_qrc-768x1365.png" alt="QR Code scanned on mobile device" width="400">
</center>

Thank you, Fausto, for making this available!



#### <a name="8"></a>Controlling the Quality of the Geometry on Custom Export

In case you had not noticed, the level of detail of tessellated surfaces generated by
the [custom exporter](http://thebuildingcoder.typepad.com/blog/about-the-author.html#5.1) can be controlled via the  `LevelOfDetail` property of the `ViewNode`, of which an instance is received in the `OnViewBegin` method of the `IExportContext` interface.

The level of detail can only be set for the entire view, not for individual elements.

It has to be so, because alternating LoD between elements would create discrepancies in tessellated meshes.

That is more than enough for today, isn't it?

Happy?
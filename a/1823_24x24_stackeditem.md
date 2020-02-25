<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</head>

<!---

- [24x24 StackedItems](https://forums.autodesk.com/t5/revit-api-forum/24x24-stackeditems/m-p/9337695)

- 10549372 [Add a new custom Ribbon Panel to a Revit built-in tab]
  https://forums.autodesk.com/t5/revit-api-forum/add-a-new-custom-ribbon-panel-to-a-revit-built-in-tab/td-p/5538772

- https://forums.autodesk.com/t5/revit-api-forum/revit-journal-file-encoding/m-p/9330501
Андрей Павлов
[apavlovY5SDS](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7264445)
Revit Journal file encoding
**Question:** What is the Revit journal file encoding?
Default filepath *C:\Users\{Username}\AppData\Local\Autodesk\Revit\Autodesk Revit 2020\Journals*.
I have trouble decoding Cyrillic characters.
**Answer:** Windows-1251, confidence 0.9824519, tested with [errepi/ude](https://github.com/errepi/ude), a C# port of the Mozilla Universal Charset Detector.

twitter:

 the #RevitAPI #DynamoBim @AutodeskForge @AutodeskRevit #bim #ForgeDevCon

Let's start the week with some ribbon button item and encoding topics
&ndash; How to create 24x24 stacked ribbon items
&ndash; Update on moving a ribbon button between panels
&ndash; Revit journal file character encoding...

linkedin:

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="100"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### 24x24 StackedItem, Ribbon Utils, Journal Encoding

Let's start the week with some ribbon button item and encoding topics:

- [How to create 24x24 stacked ribbon items](#2)
- [Update on moving a ribbon button between panels](#3)
- [Revit journal file character encoding](#4)


#### <a name="2"></a>How to Create 24x24 Stacked Ribbon Items

Jameson [jnyp](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/4918309) Nyp raised and solved this issue in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [24x24 StackedItems](https://forums.autodesk.com/t5/revit-api-forum/24x24-stackeditems/m-p/9337695):

**Question:** This may be an easy one, but so far I am struggling to find anything specific about it.
How do you make a `StackedItem` where the icons are 24x24 when there are only 2 in the stack?
It seems like it should be possible as it is used multiple times in the modify tab (see example below).

<center>
<img src="img/icon_sizes.png" alt="Icon sizes" title="Icon sizes" width="246"/>
<p style="font-size: 80%; font-style:italic">Icon sizes</p>
</center>

I have been able to set the `ShowText` property to false to get the 3 stacked icons, but when I use the same methodology with the 2 icon stack it remains 16x16 regardless of the icon resolution.
I have tried to obtain and change the button's height and width, minWidth and minHeight through the Autodesk.Window.RibbonItem object to no avail.
Has anyone had any success in creating these icons?

**Solution:** I found a solution.
In order to display the button at the 24x24 size the Autodesk.Windows.RibbonItem.Size needs to be manually set to Autodesk.Windows.RibbonItemSize.Large enum and a 24x24 icon needs to be set to the LargeImage property of the button.
I have included a code example below.
Forgive me for any poor coding techniques.
I am only a couple months into my C# developer life.
 
<pre class="code">
<span style="color:blue;">using</span>&nbsp;Autodesk.Revit.UI;
<span style="color:blue;">using</span>&nbsp;Autodesk.Windows;
<span style="color:blue;">using</span>&nbsp;System.Collections.Generic;
<span style="color:blue;">using</span>&nbsp;System.IO;
<span style="color:blue;">using</span>&nbsp;System.Reflection;
<span style="color:blue;">using</span>&nbsp;System.Windows.Media.Imaging;
<span style="color:blue;">using</span>&nbsp;YourCustomUtilityLibrary;
 
<span style="color:blue;">namespace</span>&nbsp;ReallyCoolAddin
{
&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">StackedButton</span>
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IList</span>&lt;Autodesk.Revit.RibbonItem&gt;&nbsp;Create(&nbsp;<span style="color:#2b91af;">RibbonPanel</span>&nbsp;ribbonPanel&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Get&nbsp;Assembly</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Assembly</span>&nbsp;assembly&nbsp;=&nbsp;<span style="color:#2b91af;">Assembly</span>.GetExecutingAssembly();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;assemblyLocation&nbsp;=&nbsp;assembly.Location;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Get&nbsp;DLL&nbsp;Location</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;executableLocation&nbsp;=&nbsp;<span style="color:#2b91af;">Path</span>.GetDirectoryName(&nbsp;assemblyLocation&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;dllLocationTest&nbsp;=&nbsp;<span style="color:#2b91af;">Path</span>.Combine(&nbsp;executableLocation,&nbsp;<span style="color:#a31515;">&quot;TestDLLName.dll&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Set&nbsp;Image</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BitmapSource</span>&nbsp;pb1Image&nbsp;=&nbsp;UTILImage.GetEmbeddedImage(&nbsp;assembly,&nbsp;<span style="color:#a31515;">&quot;Resources.16x16_Button1.ico&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BitmapSource</span>&nbsp;pb2Image&nbsp;=&nbsp;UTILImage.GetEmbeddedImage(&nbsp;assembly,&nbsp;<span style="color:#a31515;">&quot;Resources.16x16_Button2.ico&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BitmapSource</span>&nbsp;pb1LargeImage&nbsp;=&nbsp;UTILImage.GetEmbeddedImage(&nbsp;assembly,&nbsp;<span style="color:#a31515;">&quot;Resources.24x24_Button1.ico&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">BitmapSource</span>&nbsp;pb2LargeImage&nbsp;=&nbsp;UTILImage.GetEmbeddedImage(&nbsp;assembly,&nbsp;<span style="color:#a31515;">&quot;Resources.24x24_Button2.ico&quot;</span>&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Set&nbsp;Button&nbsp;Name</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;buttonName1&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ButtonTest1&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;buttonName2&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ButtonTest2&quot;</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Create&nbsp;push&nbsp;buttons</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">PushButtonData</span>&nbsp;buttondata1&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">PushButtonData</span>(&nbsp;buttonName1,&nbsp;buttonTextTest,&nbsp;dllLocationTest,&nbsp;<span style="color:#a31515;">&quot;Command1&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buttondata1.Image&nbsp;=&nbsp;pb1Image;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buttondata1.LargeImage&nbsp;=&nbsp;pb1LargeImage;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">PushButtonData</span>&nbsp;buttondata2&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">PushButtonData</span>(&nbsp;buttonName2,&nbsp;buttonTextTest,&nbsp;dllLocationTest,&nbsp;<span style="color:#a31515;">&quot;Command2&quot;</span>&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buttondata2.Image&nbsp;=&nbsp;pb2Image;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buttondata2.LargeImage&nbsp;=&nbsp;pb2LargeImage;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Create&nbsp;StackedItem</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IList</span>&lt;Autodesk.Revit.RibbonItem&gt;&nbsp;ribbonItem&nbsp;=&nbsp;ribbonPanel.AddStackedItems(&nbsp;buttondata1,&nbsp;buttondata2&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Find&nbsp;Autodes.Windows.RibbonItems</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UTILRibbonItem&nbsp;utilRibbon&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;UTILRibbonItem();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;btnTest1&nbsp;=&nbsp;utilRibbon.getButton(&nbsp;<span style="color:#a31515;">&quot;Tab&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;Panel&quot;</span>,&nbsp;buttonName1&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;btnTest2&nbsp;=&nbsp;utilRibbon.getButton(&nbsp;<span style="color:#a31515;">&quot;Tab&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;Panel&quot;</span>,&nbsp;buttonName2&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Set&nbsp;Size&nbsp;and&nbsp;Text&nbsp;Visibility</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;btnTest1.Size&nbsp;=&nbsp;<span style="color:#2b91af;">RibbonItemSize</span>.Large;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;btnTest1.ShowText&nbsp;=&nbsp;<span style="color:blue;">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;btnTest2.Size&nbsp;=&nbsp;<span style="color:#2b91af;">RibbonItemSize</span>.Large;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;btnTest2.ShowText&nbsp;=&nbsp;<span style="color:blue;">false</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Return&nbsp;StackedItem</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;ribbonItem;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}
</pre>

**Question:** Hi Jameson, in your code above, you use a `UTILRibbonItema` class.

What is the that?

I wasn't able to find it anywhere on the Internet.

**Answer:** The `UTILRibbonItem` class is a helper class that I use to go find RibbonItems through the Autodesk.Windows (AW) API.

It goes in to the AW and recursively searches through the tabs, panels and buttons to find the button you feed to it.

Taking all of the logic and putting it in its own class allows for easier reuse.

A larger discussion of what that class contains can be found in the other discussion thread
on [adding a new custom ribbon panel to a Revit built-in tab](https://forums.autodesk.com/t5/revit-api-forum/add-a-new-custom-ribbon-panel-to-a-revit-built-in-tab/td-p/5538772)

Here is a basic implementation:

<pre class="code">
  <span style="color:blue;">using</span>&nbsp;AW&nbsp;=&nbsp;Autodesk.Windows;
   
  <span style="color:blue;">public</span>&nbsp;AW.<span style="color:#2b91af;">RibbonItem</span>&nbsp;GetButton(&nbsp;<span style="color:blue;">string</span>&nbsp;tabName,&nbsp;
  &nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;panelName,&nbsp;<span style="color:blue;">string</span>&nbsp;itemName&nbsp;)
  {
  &nbsp;&nbsp;AW.<span style="color:#2b91af;">RibbonControl</span>&nbsp;ribbon&nbsp;=&nbsp;AW.<span style="color:#2b91af;">ComponentManager</span>.Ribbon;
  &nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;AW.<span style="color:#2b91af;">RibbonTab</span>&nbsp;tab&nbsp;<span style="color:blue;">in</span>&nbsp;ribbon.Tabs&nbsp;)
  &nbsp;&nbsp;{
  &nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;tab.Name&nbsp;==&nbsp;tabName&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;{
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;AW.<span style="color:#2b91af;">RibbonPanel</span>&nbsp;panel&nbsp;<span style="color:blue;">in</span>&nbsp;tab.Panels&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;panel.Source.Title&nbsp;==&nbsp;panelName&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;panel.FindItem(&nbsp;<span style="color:#a31515;">&quot;CustomCtrl_%CustomCtrl_%&quot;</span>&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+&nbsp;tabName&nbsp;+&nbsp;<span style="color:#a31515;">&quot;%&quot;</span>&nbsp;+&nbsp;panelName&nbsp;+&nbsp;<span style="color:#a31515;">&quot;%&quot;</span>&nbsp;+&nbsp;itemName,&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">true</span>&nbsp;)&nbsp;<span style="color:blue;">as</span>&nbsp;AW.<span style="color:#2b91af;">RibbonItem</span>;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
  &nbsp;&nbsp;&nbsp;&nbsp;}
  &nbsp;&nbsp;}
  &nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">null</span>;
  }
</pre>

Just beware that the AW API is not a documented API, so use it at your own risk as it can be changed without letting anyone know.

Many thanks to Jameson for implementing and sharing this nice clean solution, and congratulations on such prowess "only a couple months into his C# developer life"!

#### <a name="3"></a>Update on Moving a Ribbon Button Between Panels

Jameson's links above prompted me to revisit 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [adding a new custom ribbon panel to a Revit built-in tab](https://forums.autodesk.com/t5/revit-api-forum/add-a-new-custom-ribbon-panel-to-a-revit-built-in-tab/td-p/5538772) 
that I referred to here on the blog in 2014 in the article
on [moving an external command button within the ribbon](https://thebuildingcoder.typepad.com/blog/2014/07/moving-an-external-command-button-within-the-ribbon.html).

I noticed that the thread was updated after the initial publication.
Above all, the link to the sample code provided back then is no longer valid, so here is
a [local copy of it, RibbonMoveExample.zip](zip/RibbonMoveExample.zip).

Thanks again to Scott for sharing it back then.

#### <a name="4"></a>Revit Journal File Character Encoding

Finally, a quick note on the Revit journal file character encoding shared
by Андрей [apavlovY5SDS](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7264445) Павлов (Andrey Pavlov) in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [Revit journal file encoding](https://forums.autodesk.com/t5/revit-api-forum/revit-journal-file-encoding/m-p/9330501):

**Question:** What is the Revit journal file encoding?

The default file path is *C:\Users\ {Username} \AppData\Local\Autodesk\Revit\Autodesk Revit 2020\Journals*.

I have trouble decoding Cyrillic characters.

**Answer:** Windows-1251, confidence 0.9824519, tested
with [errepi/ude](https://github.com/errepi/ude),
a C# port of the Mozilla Universal Charset Detector.

Many thanks to Andrey for raising and clarifying this issue.

<!---

#### <a name="5"></a>Adjusting versus Recreating Wall Location Curve

Harald Schmidt pointed out an interesting aspect and important enhancement to the old discussion of how
to [edit wall length](https://thebuildingcoder.typepad.com/blog/2010/08/edit-wall-length.html) in
his [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [adjusting Wall.LocationCurve.Curve results in unexpected behaviour](https://forums.autodesk.com/t5/revit-api-forum/adjusting-wall-locationcurve-curve-results-in-unexpected/m-p/9328145):

**Question:** My add-in adjusts walls lines slightly (moves and rotates them a bit, and also slightly adjusts their length).

Using `wall.Location.Move` and `wall.Location.Rotate` enables adjusting the location and rotation of the walls, but not their length.

So, we decided to follow the approach suggested 10 years ago by The Building Coder to [
to [edit wall length](https://thebuildingcoder.typepad.com/blog/2010/08/edit-wall-length.html) by
creating a completely new wall location line from scratch like this:

<pre class="code">
  // get the current wall location
  LocationCurve wallLocation = myWall.Location 
    as LocationCurve;
 
  // get the points
  XYZ pt1 = wallLocation.Curve.get_EndPoint( 0 );
  XYZ pt2 = wallLocation.Curve.get_EndPoint( 1 );
 
  // change one point, e.g. move 1000 mm on X axis
 
  pt2 = pt2.Add( new XYZ( 0.01 ), 0, 0 ) );
 
  // create a new LineBound
  Line newWallLine = app.Create.NewLineBound( 
    pt1, pt2 );
 
  // update the wall curve
  wallLocation.Curve = newWallLine;
</pre>

This works fine for single walls, but fails in many cases where the wall has hosted elements like windows, and even in complex scenarios with multiple walls.

Note: the movement is always less than about some mm!

To show you what I mean, I wrote a short macro embedded in the RVT attached.

The macro `LocationLineReset` just moves the wall inside the RVT by approx. 3 mm using the method suggested by The Building Coder:

<center>
<img src="img/adjust_wall_locationcurve_curve_macros.png" alt="Adjust wall LocationCurve curve macros" title="Adjust wall LocationCurve curve macros" width="600"/> 1104
</center>

The result is the following:

<center>
<img src="img/adjust_wall_locationcurve_curve_error.png" alt="Adjust wall LocationCurve curve error" title="Adjust wall LocationCurve curve error" width="600"/> 840
</center>

This seem like an unexpected behavior.
The window is now located outside the wall, although the wall has moved only about 3mm.
Is this a bug?

If I use the second macro `ShiftWall`, which just calls `wall.LocationCurve.Move` to create an equivalent translation comparable to the former marco, the result is fine.

Is there any method to adjust the length of the wall except resetting the `wall.LocationCurve.Curve` by creating a new curve using `Line.CreateBound`? That would solve our issue as well.

**Answer:** Thank you for your interesting observation and careful analysis.

I looked at your macros and can reproduce what you say.

Besides the macro you list, `LocationLineReset`, there is another one that slightly moves the existing location line instead of creating a new one, `ShiftWall`.

That latter macro completes successfully:

<pre class="code">
  public void LocationLineReset()
  {
    doc = this.Application.ActiveUIDocument.Document;
    Wall wall = doc.GetElement( new ElementId( 305891 ) ) as Wall;
    TransactionStatus status;
    using ( Transaction trans = new Transaction( doc, "bla" ) )
    {
      trans.Start();
      LocationCurve locationCurve = wall.Location as LocationCurve;
      Line wallLine = locationCurve.Curve as Line;
      
      XYZ startPoint = wallLine.GetEndPoint(0);
      XYZ endPoint = wallLine.GetEndPoint(1);

      XYZ minimalMoveVector = new XYZ( 0.01 /* =~ 3mm*/, 0.01, 0.0);
      startPoint += minimalMoveVector;
      endPoint += minimalMoveVector;
      
      locationCurve.Curve = Line.CreateBound( startPoint, endPoint );
      status = trans.Commit();
    }
    
    if( status != TransactionStatus.Committed ) 
      MessageBox.Show( "Commit failed" );
  }
  
  public void ShiftWall()
  {
    doc = this.Application.ActiveUIDocument.Document;
    Wall wall = doc.GetElement( new ElementId( 305891 ) ) as Wall;
    TransactionStatus status;
    bool bSuccess = false;
    using ( Transaction trans = new Transaction( doc, "bla" ) )
    {
      trans.Start();

      XYZ minimalMoveVector = new XYZ( 0.01 /* =~ 3mm*/, 0.01, 0.0);				
      bSuccess = wall.Location.Move( minimalMoveVector );
      
      status = trans.Commit();
    }
    
    if( status == TransactionStatus.Committed && bSuccess) 
      MessageBox.Show( "Shift succeeded" );
  }
</pre>

I would assume that during the process of resetting the wall location line from scratch, the window position gets completely lost.

When you simply perform a small adjustment to the existing location line, the window position is retained and adjusted accordingly.

Therefore, I would suggest using the latter approach whenever possible.

You could even make use of the latter approach in several steps, adjust first one and then the other location line endpoint.

I hope this enables you to handle all required situations.


SetEndPoint method


  public void ShortenWall()
  {
    Document doc = this.Application.ActiveUIDocument.Document;
    Wall wall = doc.GetElement( new ElementId( 305891 ) ) as Wall;
    TransactionStatus status;
    using ( Transaction trans = new Transaction( doc ) )
    {
      trans.Start( "Shorten Wall");
      
      LocationCurve lc = wall.Location as LocationCurve;
      Line ll = lc.Curve as Line;
      
      double pstart = ll.GetEndParameter(0);
      double pend = ll.GetEndParameter(1);
      double pdelta = 0.05 * (pend - pstart);
      
      lc.Curve.MakeBound(pstart + pdelta, pend - pdelta); // no observable change to wall
      
      (wall.Location as LocationCurve).Curve.MakeUnbound();
      
      (wall.Location as LocationCurve).Curve.MakeBound( // no observable change to wall
        pstart + pdelta, pend - pdelta);
  
      status = trans.Commit();
    }
    
    if( status == TransactionStatus.Committed) 
      MessageBox.Show( "Shorten Wall succeeded" );
  }
  
--->

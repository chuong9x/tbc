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


#RevitAPI @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon

Here is an official answer from the Revit development team on the long-standing and recurrent issue on retrieving the path to a specific material texture bitmap file:
Question: I am working on an exporter plugin for Revit that exports all geometry from selected objects using the <code>CustomExporter</code> framework.
When extracting object materials, I can successfully get most of the information, but I can't seem to find the path to the material texture...

--->

### Material Texture Path

Here is an official answer from the Revit development team on the long-standing and recurrent issue on retrieving the path to a specific material texture bitmap file:

**Question:** I am working on an exporter plugin for Revit that exports all geometry from selected objects using the `CustomExporter` framework.

When extracting object materials, I can successfully get most of the information, but I can't seem to find the path to the material texture.

I already asked this in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on how to [extract object texture information using API](https://forums.autodesk.com/t5/revit-api-forum/extract-object-texture-information-using-api/m-p/7406055) without 
receiving any useful answer.
 
Is there a way to do this or is it a limitation in the API?

I already tried and failed to achieve this in Navisworks. Hopefully it is more easily done in Revit.
 
In Navisworks, when we add models from other CAD systems, the right textures are displayed, so why don't we have access to that same info when we try to export data?
 
We will soon have no choice but to abandon this idea and move on to something else that can support this type of workflow, because our customers are all the time asking for the textures also, and we are currently forced to do huge work by hand to provide those.
 

**Answer:** The development team responds:

The Building Coder article
on [texture bitmap and `UV` coordinates](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html) part 1
on [obtaining texture `UV` coordinates](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html#2) covers the `CustomExporter`, which is still the best way to extract information on materials and their mapping onto faces.

If you are trying to export an actual representation of an actual Revit model, you really should start with `CustomExporter`, regardless of the target format.  
 
Part 2
on [texture bitmap access](http://thebuildingcoder.typepad.com/blog/2013/07/texture-bitmap-and-uv-coordinates.html#2) says 'the texture `UV` coordinates are available, and the bitmap is not.'
 
I now realize that we have a problem here.  We've long been able to read the properties of assets, including the associated bitmaps.  The Revit API provides access to a relative path for any bitmap asset in any material. Here is a code sample demonstrating this:

<pre class="code">
&nbsp;&nbsp;<span style="color:blue;">string</span>[]&nbsp;targetMaterialNames&nbsp;=&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;A&nbsp;standard&nbsp;Revit&nbsp;material,&nbsp;with&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;textures&nbsp;in&nbsp;standard&nbsp;paths.&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;Brick,&nbsp;Common&quot;</span>,
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;A&nbsp;material&nbsp;with&nbsp;a&nbsp;single&nbsp;image&nbsp;from&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;another&nbsp;non-material&nbsp;library&nbsp;path</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;Local&nbsp;Path&nbsp;Material&quot;</span>
&nbsp;&nbsp;};
 
&nbsp;&nbsp;<span style="color:blue;">void</span>&nbsp;FindTextureBitmapPaths(&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Find&nbsp;materials</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;fec&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;doc&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;fec.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">Material</span>&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">Material</span>&gt;&nbsp;targetMaterials&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;fec.Cast&lt;<span style="color:#2b91af;">Material</span>&gt;().Where&lt;<span style="color:#2b91af;">Material</span>&gt;(&nbsp;mtl&nbsp;=&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;targetMaterialNames.Contains(&nbsp;mtl.Name&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">Material</span>&nbsp;material&nbsp;<span style="color:blue;">in</span>&nbsp;targetMaterials&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Get&nbsp;appearance&nbsp;asset&nbsp;for&nbsp;read</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ElementId</span>&nbsp;appearanceAssetId&nbsp;=&nbsp;material
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AppearanceAssetId;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">AppearanceAssetElement</span>&nbsp;appearanceAssetElem&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;doc.GetElement(&nbsp;appearanceAssetId&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">AppearanceAssetElement</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Asset</span>&nbsp;asset&nbsp;=&nbsp;appearanceAssetElem
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GetRenderingAsset();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Walk&nbsp;through&nbsp;all&nbsp;first&nbsp;level&nbsp;assets&nbsp;to&nbsp;find&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;connected&nbsp;Bitmap&nbsp;properties.&nbsp;&nbsp;Note:&nbsp;it&nbsp;is&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;possible&nbsp;to&nbsp;have&nbsp;multilevel&nbsp;connected&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;properties&nbsp;with&nbsp;Bitmaps&nbsp;in&nbsp;the&nbsp;leaf&nbsp;nodes.&nbsp;&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;So&nbsp;this&nbsp;would&nbsp;need&nbsp;to&nbsp;be&nbsp;recursive.</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;size&nbsp;=&nbsp;asset.Size;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>(&nbsp;<span style="color:blue;">int</span>&nbsp;assetIdx&nbsp;=&nbsp;0;&nbsp;assetIdx&nbsp;&lt;&nbsp;size;&nbsp;assetIdx++&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">AssetProperty</span>&nbsp;aProperty&nbsp;=&nbsp;asset[assetIdx];
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;aProperty.NumberOfConnectedProperties&nbsp;&lt;&nbsp;1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">continue</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Find&nbsp;first&nbsp;connected&nbsp;property.&nbsp;&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Should&nbsp;work&nbsp;for&nbsp;all&nbsp;current&nbsp;(2018)&nbsp;schemas.&nbsp;&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Safer&nbsp;code&nbsp;would&nbsp;loop&nbsp;through&nbsp;all&nbsp;connected</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;properties&nbsp;based&nbsp;on&nbsp;the&nbsp;number&nbsp;provided.</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Asset</span>&nbsp;connectedAsset&nbsp;=&nbsp;aProperty
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GetConnectedProperty(&nbsp;0&nbsp;)&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Asset</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;We&nbsp;are&nbsp;only&nbsp;checking&nbsp;for&nbsp;bitmap&nbsp;connected&nbsp;assets.&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;connectedAsset.Name&nbsp;==&nbsp;<span style="color:#a31515;">&quot;UnifiedBitmapSchema&quot;</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;This&nbsp;line&nbsp;is&nbsp;2018.1&nbsp;&amp;&nbsp;up&nbsp;because&nbsp;of&nbsp;the&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;property&nbsp;reference&nbsp;to&nbsp;UnifiedBitmap</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;.UnifiedbitmapBitmap.&nbsp;&nbsp;In&nbsp;earlier&nbsp;versions,</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;you&nbsp;can&nbsp;still&nbsp;reference&nbsp;the&nbsp;string&nbsp;name&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;instead:&nbsp;&quot;unifiedbitmap_Bitmap&quot;</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">AssetPropertyString</span>&nbsp;path&nbsp;=&nbsp;connectedAsset[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">UnifiedBitmap</span>.UnifiedbitmapBitmap]&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">AssetPropertyString</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;This&nbsp;will&nbsp;be&nbsp;a&nbsp;relative&nbsp;path&nbsp;to&nbsp;the&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;built&nbsp;-in&nbsp;materials&nbsp;folder,&nbsp;addiitonal&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;render&nbsp;appearance&nbsp;folder,&nbsp;or&nbsp;an&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;absolute&nbsp;path.</span>
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">TaskDialog</span>.Show(&nbsp;<span style="color:#a31515;">&quot;Connected&nbsp;bitmap&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">String</span>.Format(&nbsp;<span style="color:#a31515;">&quot;{0}&nbsp;from&nbsp;{2}:&nbsp;{1}&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;aProperty.Name,&nbsp;path.Value,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;connectedAsset.LibraryName&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
</pre>

This code provides the bitmap path.

There is still one problem, though: it is often a relative path, either to the default Materials library location, e.g. *C:\Program Files (x86)\Common Files\Autodesk Shared\Materials\Textures*, or a user-specific *Additional render appearance path*.  It could also be an absolute path if it's not in the materials folder or the additional paths.
 
I think that, to actually make this work, you will need some additional logic in your source code.  If it's an absolute path, just use it.  If it's a relative path, check if the file exists relative to the default material library installation.  If not, read the `Revit.ini` file to find the other root path(s) and check each location in turn.
 
<center>
<img src="img/rh_texture_01.png" alt="Material texture" width="275"/>
</center>

For the sake of completeness, here are pointers to some related issues:
 
- [Listing material asset textures and sub-textures](http://thebuildingcoder.typepad.com/blog/2016/10/list-material-asset-texture-and-forge-webinar-recordings.html#3)
- [Retrieve and map texture `UV` coordinates exporting geometry and material](http://thebuildingcoder.typepad.com/blog/2017/03/events-uv-coordinates-and-rooms-on-level.html#5)
- [The new Visual Materials API in Revit 2018.1](http://thebuildingcoder.typepad.com/blog/2017/08/revit-20181-and-the-visual-materials-api.html)

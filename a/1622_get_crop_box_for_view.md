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

- get crop box for a given view
  http://thebuildingcoder.typepad.com/blog/2013/09/rotating-a-plan-view.html#comment-3734421721

Efficiently retrieve crop box for given view using #RevitAPI parameter filter @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon 

...

--->

### Efficiently Retrieve Crop Box for Given View

Konrads Samulis shared a very nice solution to retrieve the crop box for a given view using a highly efficient parameter filter in
his [comment](http://thebuildingcoder.typepad.com/blog/2013/09/rotating-a-plan-view.html#comment-3734421721)
on [rotating a plan view](http://thebuildingcoder.typepad.com/blog/2013/09/rotating-a-plan-view.html).

In his own words:

In digging up this old thread, I found something quite curious in the API in 18.1, that I'm not sure was there before.

The method of using a temporary transaction (with rollback) to find the element id of the crop box was taking a very long time on a large model, so I did a bit of digging to see how I could improve it.

I noticed that in the built-in parameter `ID_PARAM` of the crop box contains the element id of the view it's in.
E.g., the crop box 'points' to the id of the view it is in using `ID_PARAM`.

That got me thinking... why not use a parameter filter to retrieve it?

So... in VB.NET (because I'm an old school vber):

<pre class="code">
<span style="color:blue;">Public</span>&nbsp;<span style="color:blue;">Function</span>&nbsp;GetELementIDOfCropBox(activeView&nbsp;<span style="color:blue;">As</span>&nbsp;View,&nbsp;revDoc&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Document</span>)&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementId</span>
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;provider&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ParameterValueProvider</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ParameterValueProvider</span>(<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementId</span>(<span style="color:blue;">CInt</span>(<span style="color:#2b91af;">BuiltInParameter</span>.ID_PARAM)))
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;ruleID_PARAM&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">FilterElementIdRule</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilterElementIdRule</span>(provider,&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilterNumericEquals</span>(),&nbsp;activeView.Id)
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;filter&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementParameterFilter</span>&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementParameterFilter</span>(ruleID_PARAM)
&nbsp;&nbsp;<span style="color:blue;">Dim</span>&nbsp;collector&nbsp;=&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(revDoc).WherePasses(filter).ToElementIds().Except(<span style="color:blue;">New</span>&nbsp;List(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">ElementId</span>)(<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementId</span>()&nbsp;{activeView.Id}))
&nbsp;&nbsp;<span style="color:blue;">Return</span>&nbsp;collector.FirstOrDefault
<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">Function</span>
</pre>

This can all be stuffed into one single statement:

<pre class="code">
<span style="color:blue;">Public</span>&nbsp;<span style="color:blue;">Function</span>&nbsp;GetELementIDOfCropBoxShortened(activeView&nbsp;<span style="color:blue;">As</span>&nbsp;View,&nbsp;revDoc&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">Document</span>)&nbsp;<span style="color:blue;">As</span>&nbsp;<span style="color:#2b91af;">ElementId</span>
&nbsp;&nbsp;<span style="color:blue;">Return</span>&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(revDoc)
&nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementParameterFilter</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilterElementIdRule</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ParameterValueProvider</span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementId</span>(<span style="color:blue;">CInt</span>(<span style="color:#2b91af;">BuiltInParameter</span>.ID_PARAM))),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">FilterNumericEquals</span>(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activeView.Id)))
&nbsp;&nbsp;&nbsp;&nbsp;.ToElementIds()
&nbsp;&nbsp;&nbsp;&nbsp;.Except(<span style="color:blue;">New</span>&nbsp;List(<span style="color:blue;">Of</span>&nbsp;<span style="color:#2b91af;">ElementId</span>)(<span style="color:blue;">New</span>&nbsp;<span style="color:#2b91af;">ElementId</span>()&nbsp;{activeView.Id})).FirstOrDefault
<span style="color:blue;">End</span>&nbsp;<span style="color:blue;">Function</span>
</pre>

Much like your original, I had to exclude the element id of the view, as it also returns its own id from `ID_PARAM`.

Hope that this may help someone &ndash; I'm sure the Dynamo people who come here for inspiration will want to know this as well...

Many thanks to Konrads for sharing this nice efficient solution!

I ported it to C# and added it 
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples) 
[release 2018.0.135.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2018.0.135.2) and
the extensive collection of filtered element examples in 
the [module CmdCollectorPerformance.cs](https://github.com/jeremytammik/the_building_coder_samples/blob/master/BuildingCoder/BuildingCoder/CmdCollectorPerformance.cs):

<pre class="code">
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;Return&nbsp;element&nbsp;id&nbsp;of&nbsp;crop&nbsp;box&nbsp;for&nbsp;a&nbsp;given&nbsp;view.</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;The&nbsp;built-in&nbsp;parameter&nbsp;ID_PARAM&nbsp;of&nbsp;the&nbsp;crop&nbsp;box&nbsp;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;contains&nbsp;the&nbsp;element&nbsp;id&nbsp;of&nbsp;the&nbsp;view&nbsp;it&nbsp;is&nbsp;used&nbsp;in;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;e.g.,&nbsp;the&nbsp;crop&nbsp;box&nbsp;&#39;points&#39;&nbsp;to&nbsp;the&nbsp;view&nbsp;using&nbsp;it&nbsp;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;via&nbsp;ID_PARAM.&nbsp;Therefore,&nbsp;we&nbsp;can&nbsp;use&nbsp;a&nbsp;parameter&nbsp;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;filter&nbsp;to&nbsp;retrieve&nbsp;all&nbsp;crop&nbsp;boxes&nbsp;with&nbsp;the&nbsp;</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;view&#39;s&nbsp;element&nbsp;id&nbsp;in&nbsp;that&nbsp;parameter.</span>
  <span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
  <span style="color:#2b91af;">ElementId</span>&nbsp;GetCropBoxFor(&nbsp;<span style="color:#2b91af;">View</span>&nbsp;view&nbsp;)
  {
  &nbsp;&nbsp;<span style="color:#2b91af;">ParameterValueProvider</span>&nbsp;provider
  &nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ParameterValueProvider</span>(&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ElementId</span>(&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span style="color:blue;">int</span>)&nbsp;<span style="color:#2b91af;">BuiltInParameter</span>.ID_PARAM&nbsp;)&nbsp;);
   
  &nbsp;&nbsp;<span style="color:#2b91af;">FilterElementIdRule</span>&nbsp;rule&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilterElementIdRule</span>(&nbsp;provider,&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilterNumericEquals</span>(),&nbsp;view.Id&nbsp;);
   
  &nbsp;&nbsp;<span style="color:#2b91af;">ElementParameterFilter</span>&nbsp;filter
  &nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ElementParameterFilter</span>(&nbsp;rule&nbsp;);
   
  &nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;view.Document&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;.WherePasses(&nbsp;filter&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;.ToElementIds()
  &nbsp;&nbsp;&nbsp;&nbsp;.Where&lt;<span style="color:#2b91af;">ElementId</span>&gt;(&nbsp;a&nbsp;=&gt;&nbsp;a.IntegerValue&nbsp;
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;!=&nbsp;view.Id.IntegerValue&nbsp;)
  &nbsp;&nbsp;&nbsp;&nbsp;.FirstOrDefault&lt;<span style="color:#2b91af;">ElementId</span>&gt;();
  }
</pre>

<center>
<img src="img/3d_view_section_box.png" alt="3D view section box" width="400"/>
</center>

<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<!--
<script src="run_prettify.js" type="text/javascript"></script>
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---


 #RevitAPI  @AutodeskRevit #bim #dynamobim @AutodeskForge #ForgeDevCon 

&ndash; 
...

--->

### Deleting a Non-Shared Project Parameter

Some happy news about two new Revit API supporter colleagues, Naveen Kumar and Zhong Wu.

They already reached second and third place in the list of top solution providers.

Here is also a very relevant recent case handled by Zhong:

- [Top Solution Authors](#2) 
- [Naveen on Naveen](#3) 
- [Zhong in FPD](#4) 
- [Zhong on Zhong](#5) 
- [Deleting a Non-Shared Project Parameter](#6) 

####<a name="2"></a> Top Solution Authors

Two of my Autodesk colleagues have joined me at the head of the list of top solution authors:

<center>
<img src="img/2018-08-26_top_solution_author.png" alt="Top solution authors" width="233"/>
</center>

Naveen joined our team in December last year and published 36 posts and  13 Solutions since May.

Zhong joined in May and already reached 88 Posts and 16 Solutions.

Welcome to the team, guys, thank you very much for your hard work, and big congratulations to both of you!


####<a name="3"></a> Naveen on Naveen

I am from Trichy, Tamilnadu and am now living in Banglore, India.

I have a degree in Mechanical Engineering from Sastra University.

After completing my B. Tech graduation, I started to learn the C# programming language and APIs for AutoCAD.

Apart from this, I am a mime Artist and love to do pencil sketching.

I enjoy watching movies and am a huge fan of music, only movie songs; my favourite composer is A.R Rahman.

####<a name="4"></a> Zhong in FPD

Zhong started working with the Revit API quite recently.

In May, he decided to make a career shift within the Forge Platform Development team and join AEC workgroup.
 
Until then, Zhong was a member of M&amp;E (multimedia and entertainment) group, leading the Maya API support.

He decided to move and focus on Forge and AEC.

He had already been helping a lot with BIM 360 and Forge evangelism at various events in China.

Having a background in Civil Engineering helps a lot in this area, of course.

Here are some words from Zhong himself reflecting on his career at Autodesk:

####<a name="5"></a> Zhong on Zhong

When you asked about the introduction, I started to look back and think over what I wanted and what I did during the past years, time flies so fast, it’s been a lot since I joined in Autodesk, the introduction seems a little too long😉.
 
I had the clear attention to join Autodesk, but during the journey, sometimes I forgot my original attention, luckily, everything seems is on track, and finally, the new opportunity and challenge comes, I am a little nervous but more exciting.  
 
 
I always remember the time, in the spring of 2005, I had the interview and got the intern offer from Autodesk when I still studied in University for my master’s degree. I was so exciting for the offer, because I got my bachelor’s degree in Civil Engineering, and during that time, computer was so popular and I believe this is something going to change the traditional industry, I am pretty interested in software development and learnt all the compute technology by myself, after graduation, instead of working as a civil engineer, I decided to continued my master degree in computer technology, so when Lang came back from Manchester to build up CADC BSD team(now is renamed to ACRD), I was so excited to be one of the team members that I believe I can finally use all my knowledge to improve the AEC industry.
 
cid:image005.jpg@01D3E6B1.22540D50

2005 summer, the 1st CADC BSD team.
 
cid:image006.jpg@01D3E6B1.22540D50

2005, October, some of Manchester team visited Shanghai ACAD BSD team
 
One year later I joined in Autodesk as a fulltime developer, I started my career on AutoCAD Architecture as a software developer, after several years late, I worked on many different Autodesk product features mainly using C++ and .NET and became the dev lead of API features. During that time, I worked with Kevin for a while in the API team, and also with Mikako to support her for ACA OMF SDK, when Autodesk start to move to cloud, I moved to another team for the new project Spiderman(which is AutoCAD on Web), still working on the API side. When Autodesk internal re-org happened on 2012, the whole team was canceled, Lang recommend me to try something new, and that comes another wonderful journey in DevTech M&E team.
 
ADN is an amazing family, it’s totally different from engineering team I worked before. For me, I like this more because I can have direct contact with industry guys and can learn/help them directly. And M&E was a very interesting and fantasy industry with lots of cool technology, I feel so lucky to be one member of the small global team, I learnt the API of Maya and then Motionbuilder and work closely with different groups including engineering team and sales team to help partners, then also take the leadership for Maya API support. Everything is going so well that sometimes I forgot about my initial idea to join Autodesk.   
 
cid:image007.jpg@01D3E6B1.22540D50

My 1st ADN Devtech team meeting in San Rafel in 2013
 
cid:image009.jpg@01D3E6B1.22540D50

Devtec M&E Team meeting & FITC conference in Toronto in 2018
 
When Forge comes to our team, it’s totally a new opportunity to expand our technique stack and also the industry stack, web-based technology is quite different from the desktop software, it gives you some pain at first, but you will enjoy it more when you get involved. I made my decision to embrace and learn the technology, I also connected lots of developers during all kinds of presentation, workshop, conferences .etc. many of them are from AEC industry, this reminds me the initial idea to choose Autodesk as my 1st job, I started to learn BIM360 and try to involve more with AEC industry intended, I strongly believe this is the industry that I can contribute more.
 
Now you know all my previous story, when latest re-org comes to have more industry focus, also inspired by Stephen’s career development, 1st idea comes to my mind is that I have to make my decision for my own career path, I’d like to focus more on AEC industry, to improve the efficiency of AEC industry by using model technology.


####<a name="6"></a> Deleting a Non-Shared Project Parameter

One of Zhong's recent cases answers the question raised in
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [`BindingMap.Remove` being ineffective on a non-shared project parameter]( https://forums.autodesk.com/t5/revit-api-forum/deleting-a-non-shared-project-parameter/td-p/5975020):

The partner is complaining about an issue deleting non-shared project parameter, and there is no information in the SDK help doc to mention this.

The discussion
on [deleting a shared parameter](http://thebuildingcoder.typepad.com/blog/2009/08/deleting-a-shared-parameter.html) shows
how to delete a shared parameter, but I could not find the proper way to delete a non-shared project parameter.

I checked with the Revit engineering team to see if they can fix this or this is by design.

They responded:

> Yes, that’s a known issue and logged under the number REVIT-136670.

> As a workaround, you can delete a non-shared project parameter by retrieving the associated `ParameterElement`.

> If you delete that element, the non-shared parameter will also be removed. 

I verified this with the following sample code:
 
<pre class="code">
&nbsp;&nbsp;<span style="color:blue;">void</span>&nbsp;DeleteNonSharedProjectParam(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Document</span>&nbsp;doc,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;parametername&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>&nbsp;ps
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;doc&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">ParameterElement</span>&nbsp;)&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ParameterElement</span>&nbsp;projectparameter&nbsp;=&nbsp;<span style="color:blue;">null</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>(&nbsp;<span style="color:#2b91af;">ParameterElement</span>&nbsp;pe&nbsp;<span style="color:blue;">in</span>&nbsp;ps&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;pe.GetDefinition().Name.Equals(&nbsp;parametername&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;projectparameter&nbsp;=&nbsp;pe;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;projectparameter&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;doc.Delete(&nbsp;projectparameter.Id&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
</pre>

It works as expected.

I hope this helps.

Many thanks again to Zhong for his clear analysis and nice solution.

I added Zhong's solution to
to [The Building Coder Samples](https://github.com/jeremytammik/the_building_coder_samples) 
in [release 2019.0.143.2](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.143.2).

You can see the changes I made by looking at
the [diff to the preceding release](https://github.com/jeremytammik/the_building_coder_samples/compare/2019.0.143.1...2019.0.143.2).

I later noticed that Zhong's code can be shortened a little bit further like this making use of a LINQ `Where` clause:

<pre class="code">
&nbsp;&nbsp;<span style="color:#2b91af;">ParameterElement</span>&nbsp;projectparameter
&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">FilteredElementCollector</span>(&nbsp;doc&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WhereElementIsNotElementType()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.OfClass(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">ParameterElement</span>&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Cast&lt;<span style="color:#2b91af;">ParameterElement</span>&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Where(&nbsp;e&nbsp;=&gt;&nbsp;e.GetDefinition()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Name.Equals(&nbsp;parametername&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.FirstOrDefault();
 
&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;projectparameter&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;doc.Delete(&nbsp;projectparameter.Id&nbsp;);
&nbsp;&nbsp;}
</pre>

The modification is preserved
in [The Building Coder samples release 2019.0.143.3](https://github.com/jeremytammik/the_building_coder_samples/releases/tag/2019.0.143.3).


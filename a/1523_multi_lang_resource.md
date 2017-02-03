<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="run_prettify.js" type="text/javascript"></script>
<!--
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js" type="text/javascript"></script>
-->
</head>

<!---

- 12610584 [Why the '/language' key switches CurrentCulture instead of CurrentUICulture?]
  http://forums.autodesk.com/t5/revit-api-forum/why-the-language-key-switches-currentculture-instead-of/m-p/6837625

- march accelerator in SF is at risk, please sign up
  accelerator in gothenburg and barcelona

#RevitAPI @AutodeskRevit #aec #bim #dynamobim @AutodeskForge

&ndash; 
...

-->

### Supporting Multiple Language Resource Files


- [Supporting multiple language resource files](#2)
- [Upcoming Forge accelerators](#3)
- [Find label or tag containing specific shared parameter](#4)


#### <a name="2"></a>Supporting Multiple Language Resource Files

Andrey Bushman raised another interesting issue in 
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [why the '/language' key switches `CurrentCulture` instead of `CurrentUICulture`](http://forums.autodesk.com/t5/revit-api-forum/why-the-language-key-switches-currentculture-instead-of/m-p/6837625)

In the course of our discussion, Andrey pointed out his highly interesting blog and shared a fully functional Revit add-in demonstrating how to support multi-language resources with all conceivable frills, bells and whistles:

- [Andrey's Revit programming notes blog](https://revit-addins.blogspot.ru/2017/01/revit-201711.html) (in Russian)
- [RevitMultiLanguageAddInExample GitHub repository](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample) (in C# .NET)

If, like me, your command of Russian is limited, it helps to switch
on [automatic translation](https://chrome.google.com/webstore/detail/google-translate/aapbdbdomjkkjkaonfhkkikfgjllcleb) for
the Russian blog  :-)

By the way, this add-in obviously also makes use of
Andrey's [NuGet Revit API package](http://thebuildingcoder.typepad.com/blog/2016/12/nuget-revit-api-package.html),
now updated to support the recent additional Revit 2017.X.Y releases.

Andrey's reason for raising the thread in the first place was a weire behaviour setting the UI culture in Revit 2017.1.1, which I passed on to the development team for further exploration.

However, Andrey provides a workaround for that too, in the module 
[RevitPatches.cs](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample/blob/master/RevitMultiLanguageAddInExample/RevitPatches.cs):

<pre class="code">
span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">RevitPatches</span>
{
&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;This&nbsp;patch&nbsp;fixes&nbsp;the&nbsp;bug&nbsp;of&nbsp;localization&nbsp;switching</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;for&nbsp;Revit&nbsp;2017.1.1.&nbsp;It&nbsp;switches&nbsp;the&nbsp;&#39;Thread</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;.CurrentThread.CurrentUICulture&#39;&nbsp;and&nbsp;&#39;Thread</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;.CurrentThread.CurrentCulture&#39;&nbsp;properties&nbsp;according&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;the&nbsp;UI&nbsp;localization&nbsp;of&nbsp;Revit&nbsp;current&nbsp;session.</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;/</span><span style="color:gray;">summary</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:gray;">///</span><span style="color:green;">&nbsp;</span><span style="color:gray;">&lt;</span><span style="color:gray;">param</span><span style="color:gray;">&nbsp;name</span><span style="color:gray;">=</span><span style="color:gray;">&quot;</span>lang<span style="color:gray;">&quot;</span><span style="color:gray;">&gt;</span><span style="color:green;">The&nbsp;target&nbsp;language.</span><span style="color:gray;">&lt;/</span><span style="color:gray;">param</span><span style="color:gray;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;PatchCultures(&nbsp;<span style="color:#2b91af;">LanguageType</span>&nbsp;lang&nbsp;)
&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(&nbsp;!<span style="color:#2b91af;">Enum</span>.IsDefined(&nbsp;<span style="color:blue;">typeof</span>(&nbsp;<span style="color:#2b91af;">LanguageType</span>&nbsp;),&nbsp;lang&nbsp;)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">throw</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArgumentException</span>(&nbsp;<span style="color:blue;">nameof</span>(&nbsp;lang&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;language;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">switch</span>(&nbsp;lang&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Unknown:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.English_USA:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;en-US&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.German:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;de-DE&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Spanish:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;es-ES&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.French:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;fr-FR&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Italian:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;it-IT&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Dutch:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;nl-BE&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Chinese_Simplified:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;zh-CHS&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Chinese_Traditional:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;zh-CHT&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Japanese:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ja-JP&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Korean:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ko-KR&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Russian:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ru-RU&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Czech:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;cs-CZ&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Polish:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;pl-PL&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Hungarian:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;hu-HU&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">LanguageType</span>.Brazilian_Portuguese:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;language&nbsp;=&nbsp;<span style="color:#a31515;">&quot;pt-BR&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">default</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Maybe&nbsp;new&nbsp;value&nbsp;of&nbsp;the&nbsp;enum&nbsp;hasn&#39;t&nbsp;own&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;`case`...</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">throw</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArgumentException</span>(&nbsp;<span style="color:blue;">nameof</span>(&nbsp;lang&nbsp;)&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">CultureInfo</span>&nbsp;ui_culture&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">CultureInfo</span>(&nbsp;language&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">CultureInfo</span>&nbsp;culture&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">CultureInfo</span>(&nbsp;language&nbsp;);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Thread</span>.CurrentThread.CurrentUICulture&nbsp;=&nbsp;ui_culture;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Thread</span>.CurrentThread.CurrentCulture&nbsp;=&nbsp;culture;
&nbsp;&nbsp;}
}
</pre>

Here is a summary of our discussion of this:

Revit 2017.1.1
 
My external application (add-in) has two localizations: English (by default) and Russian. I saw that my external application (add-in) on the some computers uses wrong localization. One of that computers has a 'clear' Revit without installed custom add-ins.
 
If I launch `revit.exe` with the `/language RUS` command line option, then the Revit UI is Russian, but my add-in  UI still uses the default localization (i.e. "en") instead of "ru".
 
I extract the localized resources through the `ResourcesManager` class in my code. That is the native way to work with resources in .NET. I know that `ResourcesManager` chooses the localized resources based on the value of  `Thread.CurrentThread.CurrentUICulture`.
 
Therefore, I assumed that this value isn't `"ru"` or `"ru-RU"`, for some reason, despite the `/language RUS` key. I checked and confirmed that assumption (look my code below):  the `Thread.CurrentThread.CurrentUICulture` property is set to `"en"` instead of `"ru"`. Also, I see that `Thread.CurrentThread.CurrentCulture` uses `"ru-RU"` instead of `"en"`.
 
Why does the `/language` key not switch the setting of `Thread.CurrentThread.CurrentUICulture` responsible for the user interface (UI)?

Is it bug?

<pre class="code">
  Result IExternalApplication.OnStartup(UIControlledApplication application)
  {
    ...
    // I use the `/language RUS` key for revit.exe
    // But I see that my add-in user the 'default' localization
    // instead of 'ru'. Hm...
    // Ok,I will check the CurrentCulture and CurrentUICulture 
    // values...
    //
    // Oops... Localization was changed by the `/language RUS` 
    // key, but not for that thread which shall to have this 
    // change! Why???
    CultureInfo n = Thread.CurrentThread.CurrentCulture; // ru-RU
    CultureInfo m = Thread.CurrentThread.CurrentUICulture; // en
  
    // I can fix this problem myself:
    // The CurrentUICulture switching fixes the problem, but 
    // why such problem occurs in Revit?
    CultureInfo k = new CultureInfo("ru");
    Thread.CurrentThread.CurrentUICulture = k;
    
    // Now my add-in uses right localization. 
    ...
  }
</pre>

Later, I found that this problem exists in Revit 2017.1.1, but this problem is absent in Revit 2017.
 
For demonstration of this problem I created the project and published it on bitbucket,
in [Andrey-Bushman/research_of_revit_culture_switching_problem](https://bitbucket.org/Andrey-Bushman/research_of_revit_culture_switching_problem).

You can fix this problem in Revit 2017.1.1 by using a class like this:

<pre class="code">
  public sealed class UICultureSwitcher : IDisposable {
  
    CultureInfo previous;
    
    public UICultureSwitcher() {
  
      CultureInfo culture = new CultureInfo(Thread
        .CurrentThread.CurrentCulture.Name);
  
      previous = Thread.CurrentThread.CurrentUICulture;
      Thread.CurrentThread.CurrentUICulture = culture;
    }
  
    void IDisposable.Dispose() {
      Thread.CurrentThread.CurrentUICulture = previous;
    }
  }
</pre>

Place code of methods of each external command and external application inside a `using` block with this class member initializing, and the problem will be fixed. My monologue is ended.

Expanded info and the patch are provided in the [Russian blog post on Revit 2017.1.1](https://revit-addins.blogspot.ru/2017/01/revit-201711.html).

I also created the [RevitMultiLanguageAddInExample](https://github.com/Andrey-Bushman/RevitMultiLanguageAddInExample) simple example and published it on GitHub here:

I made an [eight-minute video on resx using](https://www.youtube.com/watch?v=DKCm3p9Gp9M) which shows the easiest way of creating and editing resource files:

<center>
<iframe width="480" height="270" src="https://www.youtube.com/embed/DKCm3p9Gp9M?rel=0" frameborder="0" allowfullscreen></iframe>
</center>

When you use the `/language RUS` key the UI of my add-in, its tooltip, expanded tooltip, and help file displayed on pressing F1 are in Russian.

When you use `/language ENU` key the UI of my add-in, its tooltip, expanded tooltip, and help file displayed on pressing F1 are in English.
 
The same applies for the TaskDialog content.

Many thanks to Andrey for this beautiful and illuminating sample!
 

#### <a name="3"></a>Upcoming Forge Accelerators

We have a number
of [Forge accelerators](http://autodeskcloudaccelerator.com/) coming up in
the [next couple of months](http://autodeskcloudaccelerator.com/prague-2/):

- San Francisco, USA &ndash; March 6-10
- Gothenburg, Sweden &ndash; March 27-30
- Barcelona, Spain &ndash; June 12-16

<center>
<img src="img/2017-02_upcoming_accelerators.png" alt="Upcoming Forge accelerators" width="668"/>
</center>

I am planning on attending the two European ones and would love to see you there too.

Before that, however, the San Fransisco accelerator provides the very next chance to attend &ndash; and the early bird get the worm &ndash; so grab you chance while you can!

<center>
<img src="img/bird_with_worm.png" alt="Bird with worm" width="183"/>
</center>



#### <a name="4"></a>Find Label or Tag Containing Specific Shared Parameter

[find label or tag containing specific shared parameter](http://forums.autodesk.com/t5/revit-api-forum/find-label-tag-containing-specific-shared-parameter/m-p/6834034)

**Question:** 

**Answer:** 

<pre class="code">
</pre>



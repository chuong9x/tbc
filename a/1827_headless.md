<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<link rel="stylesheet" type="text/css" href="bc.css">
<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js" type="text/javascript"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</head>

<!---


twitter:

Two short notes on splitting pipes retaining connections and headless Revit, launching it with no user interface with the #RevitAPI #DynamoBim @AutodeskForge @AutodeskRevit #bim #ForgeDevCon http://bit.ly/rvtheadless

Two short notes on splitting pipes retaining connections and headless Revit, launching it with no user interface...

&ndash; 
...

linkedin:

#bim #DynamoBim #ForgeDevCon #Revit #API #IFC #SDK #AI #VisualStudio #Autodesk #AEC #adsk

the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread

<center>
<img src="img/" alt="" title="" width="100"/>
<p style="font-size: 80%; font-style:italic"></p>
</center>

-->

### Split Pipe and Headless Revit

Two short notes on splitting pipes and headless Revit:

- [Headless Revit](#2)
- [Split pipe retaining connections](#3)

#### <a name="2"></a> Headless Revit

Kennan Chen has made several exciting contributions recently and mentions yet another one in his answer to
the [Revit API discussion forum](http://forums.autodesk.com/t5/revit-api-forum/bd-p/160) thread
on [family related memory leaks](https://forums.autodesk.com/t5/revit-api-forum/family-related-memory-leaks/m-p/8738515):

> A possible alternative to handle memory leaks is to run Revit in headless mode.
Contrary to the documented approach to create an add-in, you can start an application which hosts a Revit runtime within the same process which enables you to do what you want with the top-level `Application` object.
Just like Navisworks.

> In your case, for each project, start a headless Revit to finish your process, then close the application.
The problem is, for each project, there is a waste of boot time.
Since headless Revit don't start the renderer and anything about the UI, less memory will be consumed to make it possible to run several tasks in parallel on your machine.
Moreover, you can even set up a cluster to handle tons of projects in parallel which I believe is the key to enable Forge to resolve Revit files in cloud.
As to how to set up a headless Revit, find a file named `lcldrevit.dll` or `lcrvtutil.dll` (newer version of Navisworks) under {Navisworks root folder}\Loaders\Rx folder.
`LcRevitLoad.DoInit` contains all you need to start your own headless Revit.
To make things easier still, I created a library called Revit.Headless to do all that loading logic for you.

> Visit the [Revit.Headless GitHub repository](https://github.com/KennanChan/Revit.Headless) for more details.
Also available via NuGet.

Many thanks to Kennan for researching and sharing this exciting possibility!

#### <a name="3"></a> Split Pipe Retaining Connections

A pretty standard functionality in the Revit MEP user interface can be a bit tricky to find in the API:

**Question:** The UI provides the split command (SL) to split a pipe into two without losing other connected elements.
How can I achieve the same in API?

**Answer:** You can use the `PlumbingUtils` `BreakCurve` method.
This is also available for duct work in `MechanicalUtils` `BreakCurve`.

<center>
<img src="img/split_pipe_retaining_connections.jpg" alt="Split pipe retaining connections" title="Split pipe retaining connections" width="481"/>
</center>

